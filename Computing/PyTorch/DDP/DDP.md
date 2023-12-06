These notes were primarily based upon the [DDP publication on Arxiv](https://arxiv.org/pdf/2006.15704.pdf).

Distributed data parallelism, broadly speaking, replicates a model on every computational resource to generate gradients independently. Gradients are then communicated at each iteration to keep model replicas consistent.

This is conceptually simple, but subtle dependencies between computation and communication make it non-trivial to optimise. As of v1.5, PyTorch natively provided several techniques to accelerate DDP, including **bucketing gradients**, **overlapping computation with communication**, and **skipping gradient synchronisation**.
```toc
```
# Introduction
On a high level, the only significant change made by DDP to standard neural network training is a synchronisation of gradients (or parameters, depending on the algorithm). However, squeezing out the last bit of performance takes a large design effort.

# Background 
## PyTorch
This section gives a high-level summary of PyTorch, which I actually found quite fun to read.

PyTorch organises values into `Tensor` objects, which are generic $n$-dimensional arrays with an associated set of data manipulating operations. A `Module` defines a transform from input values to output values, and its behaviour during the forward pass is specified by its `forward` member function. A `Module` can contains `Tensor` objects as parameters. 

An application composes its own `Module` by stitching together native `Module` objects, and `Function`s (e.g. relu, pool, etc) in the custom `forward` function.

A typical training step consists of 3 stages:

1. During the forward pass, PyTorch builds an autograd graph to record actions performed. 
2. In the backward pass, it uses the autograd graph to conduct backpropagation to generate gradients. 
3. The optimizer applies the gradients to update parameters.

## AllReduce
`AllReduce` is the primitive communication API used by DDP to sum gradients across all processes. The `AllReduce` operation expects each participating process to provide an equally-sized tensor, collectively applies some arithmetic operation (e.g. `sum`, `prod`, `min`) to inputs tensors from all processes, and returns the same result tensor to each participant.

A naive implementation could simply let every process broadcast its input tensor to all peers and then apply the arithmetic operation independently. However, more efficient algorithms exist, such as **ring-based `AllReduce`** and **tree-based `AllReduce`.**

Since one `AllReduce` operation cannot start until all processes join, it is considered to be a *synchronized* communication.

# System Design
## Gradient Reduction
The gradient reduction algorithm in DDP has evolved over time. To understand the latest implementation, we start from a naive solution and gradually introduce more complexities.

### A Naive Solution
DDP guarantees correctness by letting all training processes (1) start from the same model state, and (2) consume the same gradients in every iteration.

To implement the latter, a naive solution can insert a gradient synchronisation phase after the local backward pass and before updated local parameters[^fn1].

The DDP API does not provide an explicit entry point for this phase as there is nothing between `loss.backward()` and `model.step()`. 
Instead, DDP can register autograd hooks to trigger computation after every backward pass. When fired, each hook scans through all local model parameters, and retrieves the gradient tensor for each parameter. It then uses `AllReduce` to calculate the average gradients on each parameter across all processes, and writes the result back to the gradient tensor.

This naive solution is sufficient, but there are 2 performance limitations:

1. Collective communication (with `AllReduce` ) performs poorly on small tensors;
2. Separating gradient computation and synchronisation means we can't overlap communication with computation.

The following sections describe solutions to these concerns.

### Gradient Bucketing
Gradient bucketing is motivated by the observation that collective communications are more efficient on large tensors. 

Instead of launching an `AllReduce` immediately when each gradient tensor becomes available, DDP can achieve better throughput and lower latency if it waits for a short period of time and buckets multiple gradients into one `AllReduce`. This maximises bandwidth utilisation, and would be especially helpful for models with many small parameters.

However, DDP should not communicate all gradients in a single `AllReduce`, otherwise no communication can start before the computation is over.

### Overlap Communication with Computation
The `AllReduce` operation on gradients can start before the local backward pass finishes. With bucketing, DDP only needs to wait for all contents in the same bucket before launching communications. Under such settings, triggering an `AllReduce` at the end of the backward pass is no longer sufficient. Instead, `AllReduce` needs to happen more frequently.

To understand this, I found it useful to recall the animations on the [gradient checkpointing GitHub](https://github.com/cybertronai/gradient-checkpointing). These show how gradients are computed and used to update parameters.  The key point is that gradient are communicated - and parameters subsequently updated - before the entire backwards pass is complete. ^b60279

There is significantly more detail in this section in the paper.

### Gradient Accumulation
Instead of launching `AllReduce` in every iteration, it is possible to conduct $n$ local training iterations before synchronizing gradients globally. I originally understood this as being helpful if the input batch is too large to fit on one device, but it also helps to reduce communication overhead.

This does to some degree conflict with the gradient reduction algorithm used in DDP. Parameters that are unused in the forward pass would be marked as ready (for backprop) at the end of every forward pass, but they could still be used in susequent iterations. Moreover, DDP cannot tell whether the application plans to immediately invoke `optimizer.step()` after `loss.backward()`, or accumulate gradients through multiple iterations. This might result in unncessary communication.

Therefore, DDP introduces the `no_sync()` interface for this use case. Below is an example code snippet:

![](_attachments/Screenshot%202022-08-21%20at%2018.53.03.png)

Under the hood, the implementation for `no_sync` is very simple. The context manager just toggles a flag on entering and exiting the context, and the flag is consumed int he `forward` function of DDP. In `no_sync` mode, all DDP hooks are disabled and the first backward pass out of the context will synchronise the accumulated gradients together. 

# Buffer Broadcast
One subtlety in DDP is how it synchronises buffers between ranks. 

DDP [broadcasts](NCCL%20Collectives#Broadcast) buffers from rank 0 to all other processes. This means that the buffers are only updated according to the data seen on rank 0; all other data is ignored. For Speechmatics, this is relevant in the context of GlobalNorm buffers.

In the [PyTorch API](https://pytorch.org/docs/stable/generated/torch.nn.parallel.DistributedDataParallel.html), there is a boolean argument called `broadcast_buffers` which controls this behaviour; we can turn off the broadcasting if we want to. 

[^fn1]: This pretty much describes my understanding prior to reading this article.

