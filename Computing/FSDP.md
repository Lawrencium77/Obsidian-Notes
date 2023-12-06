These notes are based on [this](https://engineering.fb.com/2021/07/15/open-source/fsdp/) Facebook blog.

```toc
```

## Description

**Fully-Sharded Data Parallel** is a technique to fit larger models on GPUs. In this sense, it is like [Tensor Parallel](../ML/Tensor%20Parallel.md) or [pipeline parallel](Engineering%20Challenges%20in%20Training%20Massive%20Transformers.md). Except that it's [not a form of model parallel](#^379a2e).

In FSDP, each GPU contains a shard of a model. Then, **for each layer**: 

* In the forward pass, an [AllGather](NCCL%20Collectives.md) is used such that each device has a full copy of the layer's weights. 
* This gathering of the weights is performed again before the backward pass.
* After the backward pass, the local gradients are averaged and sharded across GPUs by means of a reduce-scatter. This means that each device has only its component of the gradient tensor for the whole layer.

![](_attachments/Screenshot%202023-04-25%20at%2016.33.57.png)

To maximise memory efficiency, we discard the full weights after each layer's forward pass. 

Here's a nice pseudo-code summary:

```
Forward Pass:
	for layer_i in layers:
		all-gather full_weights for layer_i
		forward pass for layer_i
		discard full weights for layer_i

Backward Pass:
	for layer_i in layers:
		all-gather full weights for layer_i
		backward pass for layer_i
		discard full weights for layer_i
		reduce-scatter gradients for layer_i
```

Each GPU does a forward and backward pass for the whole model, on a different piece of data. So it's **not** model parallel. ^379a2e

## Comparison With DDP
During the backwards pass in [DDP](PyTorch/DDP/DDP.md), we perform an **AllReduce** of the gradients. This consists of a **ReduceScatter**, and an **AllGather**.

The FSDP backwards pass consists of an **AllGather** of our weights, and then a **ReduceScatter** of the gradients. 
In other words, the steps are reversed.  But the amount of communication (i.e. bytes of data transferred) is the same.[^fn1]

[^fn1]: The reason for this is obvious: the sizes of the parameter and gradient tensors are identical. So which one we are transmitting doesn't matter - the number of bytes transferred will be the same.