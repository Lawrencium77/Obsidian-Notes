These notes are based upon [this NVIDIA blog](https://docs.nvidia.com/deeplearning/transformer-engine/user-guide/examples/fp8_primer.html). 

### Introduction to FP8
H100s introduce support for FP8. This enables higher throughput of matmuls and convolutions.

There are two types of FP8, **E4M3** and **E5M2**. These are shown below:

![](_attachments/Screenshot%202023-04-07%20at%2011.31.11.png)

Both types of FP8 may be used during neural network training:

> * Activations & weights typically require more precision $\implies$use E4M3
> * Gradients typically require higher dynamic range $\implies$ use E5M2

### Recap of Mixed Precision Training
Let's remind ourselves of how mixed precision training works.

In FP16, there are two stages in mixed precision training: choosing which operations should be performed in FP16, and dynamic loss scaling:

> * Choosing which operations are performed in FP16 requires analysis of their numerical behaviour. Matmuls, convolutions, and normalization are safe whereas `exp` and `norm` operations require high precision.
> * Dynamic loss scaling is used to avoid both over- and underflow of the gradients. These may occur since, while the dynamic range of FP16 is large enough to store the gradients, the distribution may be centred around values that are too high or too low. Scaling the loss shifts those distributions.
> * Loss scaling uses **powers of 2**, to prevent any "changes to the numerics".

![](_attachments/Screenshot%202023-04-07%20at%2011.36.09.png)

### Mixed Precision Training with FP8
While the dynamic range of the FP8 types is sufficient to store any particular activation or gradient, it is not sufficient for all of them at the same time. This makes the single loss scaling factor strategy, which worked for FP16, infeasible for FP8.

Instead, we use distinct scaling factors for each FP8 tensor.

There are multiple strategies to determine a scale factor for a given tensor:

> * **JIT Scaling**: We choose the scaling factor based on the `amax` of the tensor being produced. In practice, this has high overhead which diminishes the gains from using FP8.
> * **Delayed Scaling**: This strategy chooses the scaling factor based on the `amax` of tensors seen in past iterations. This enables full performance of FP8 but requires storing the history of `amax` values as additional parameters of the FP8 operators. 

![](_attachments/Screenshot%202023-04-07%20at%2011.41.29.png)

This certainly has a lot of overlap with [dynamic and static quantization](Quantization%20in%20Neural%20Nets%20-%20Key%20Concepts#D.%20Range%20Calibration%20Algorithms:%20Static%20vs%20Dynamic%20Quantisation).

### Transformer Engine
I only picked out the interesting parts from the section.

#### Handling Backward Pass
When a model is run FP8 in multi-GPU training, some communication is required to synchronise scaling factors and `amax` history. 
To do so without too much overhead, NVIDIA's implementation aggregates the tensors before performing the communication. (I assume this is like [Ring-Based AllReduce](../Computing/Ring-Based%20AllReduce.md)).



