These notes describe how QDQ nodes are used to quantise neural nets. These ideas are very strongly related to the TensorRT implementation but I added a bit of my own insight.
```toc
```

## QDQ Nodes
When doing PTQ or QAT in TensorRT, we insert QDQ nodes into the input & weight for every linear/conv layer. For example:

![](_attachments/Screenshot%202022-11-16%20at%2013.37.18.png)

We calibrate in this setup. In other words, we measure the distributions of our input tensors and tune our quantisation clipping range accordingly.

But once this process is complete, we are not executing INT8 matmuls. We're doing a QDQ op, and then executing in FP32. How do we convert this graph into one that's actually running quantised matmuls?

## Processing of QDQ Networks
TensorRT expects a QDQ layer pair on each of the inputs of a quantisable layer. During network optimisation, it moves Q/DQ nodes in a layer called QDQ **propagation**. The goal is to maximise the proportion of the graph that can be processed at low precision.
To be clear: Q nodes are propagated backwards, and DQ nodes are propagated forwards.

Q layers swap places with operations that commute with quantisation. And DQ layers swap places with layers that commute with dequantisation.

### Mathematical Equivalence of DQ Propagation Through Linear Layers

This all makes sense. But when I first learnt this, I was confused - which layers commute  with Q/DQ nodes? Do we actually end up running matmuls in INT8?

The answer is yes. To understand this, let's consider what happens for a linear layer. Take a matmul $Y=XW$. When quantising:

$$Y_{ij}=\sum_kX_{ik}W_{kj} =\sum_kS_XX_{q,ik}\cdot S_WW_{q,kj} = S_XS_W\sum_kX_{q,ik}W_{q,kj}$$
where $S_X, S_W$ are quantisation scaling factors. Hence:

$$Y_{ij}=S_XS_W\sum_kX_{q,ik}W_{q,kj}$$
The term $\sum_kX_{q,ik}W_{q,kj}$ corresponds to an INT8 matmul. This equation tells us that the dequantisation operator (i.e. the DQ node) is simply a *scaling*. We accumulate the result in INT32, and the cast to FP32 with this multiplication. Moreover, it shows that **dequantisating an INT8 matmul is exactly equivalent to dequantising, and then doing an FP32 matmul.**
This corresponds to INT8 mode 1 in [FasterTransformer's BERT implementation](https://github.com/NVIDIA/FasterTransformer/blob/main/docs/bert_guide.md#model-architecture).

It can be more performant to ensure our GEMM produces an INT8 (instead of INT32) output. This makes our DRAM accesses even cheaper.
This requires [requantisation](A%20More%20Practical%20Perspective.md#^b632b5):

$$Y_{ij}=S_XS_W\sum_kX_{q,ik}W_{q,kj}=S_YY_{q,ij}$$

Hence:

$$Y_{q,ij}=\frac{S_XS_W}{S_Y}\sum_kX_{q,ik}W_{q,kj}$$

I'm assuming there is a clipping operation here as well. This corresponds to INT8 mode 2 in [FasterTransformer's BERT implementation](https://github.com/NVIDIA/FasterTransformer/blob/main/docs/bert_guide.md#model-architecture). This is more performant than mode 1, but sacrifices some accuracy.

> [!INFO]
> The key point here is that we can propagate DQ nodes through a matmul. This explains why measuring model performance (and doing calibration, QAT, etc.) with QDQ nodes is valid; it's *exactly* the same as when we move the DQ node to the other side of the matmul, and execute in INT8.
> Another example of an operation that is commutable with dequantisation is $\max$ or $\textrm{maxpool}$.

> [!Question]
> I'm assuming that the $S_Y$ scale is obtained during calibration as well (for INT8 mode 2). This would involve putting a QDQ node *after* each matmul during calibration. Is this correct?

## Examining FasterTransformer's BERT
I've found it useful to look closely at the ordering of QDQ nodes in FasterTransformer's BERT implementation. Specifically, we examine the operations surrounding their fused Bias + GELU kernel.

In Mode 1:

![](_attachments/Screenshot%202022-11-16%20at%2014.05.04.png)

and in Mode 2:

![](_attachments/Screenshot%202022-11-16%20at%2014.04.47.png)

$X$ and $W$ represent the input vector and parameters respectively. $S_X, S_W, S_Y$ and $S_{\textrm{out}}$ represent quantisation scale factors. The arrow colouring represents the datatype of each operation output. Blue dashed boxes indicate fused kernels.

Examining these graphs allows us to understand exactly where quantisation and dequantisation is happening through a series of operations. In both cases, we avoid expensive FP32 DRAM reads/writes.