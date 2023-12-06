These notes were made based upon [this talk](https://drive.google.com/file/d/1L3Ivia86iuv4d9nTducQyHYABjoTcGBP/view) by a guy who works at Nvidia.  It relates very strongly to a blog by Lilian Weng, which I also made notes on: [How to Train Really Large Models on Many GPUs](How%20to%20Train%20Really%20Large%20Models%20on%20Many%20GPUs.md)

## Overview
1. Mixed Precision Training with float16 and bfloat16
2. Accelerating layernorm, dropout and activation functions
3. Parallelism Nomenclature 
4. Model and data parallelism:
	1. Distributed data parallelism and chunkwise asynchronous gradient allreduce
	2. Tensor Parallelism
	3. Pipeline Parallelism
	4. Sequence parallelism of layernorm and dropout*
	5. Selective Activation recomputation*

\* I don't really understand these.

## Mixed Precision Training
There are two quantities used in describing the properties of floating point numbers:

* **Precision**: Defined as the smallest value $\epsilon$ such that $1+\epsilon$ can be represented. This is $2^{-f}$, where $f$ is the number of significand bits.
* **Dynamic Range**: Defined as $log_{10}$ of the largest to smallest representable positive numbers

bfloat16 trades off precision for dynamic range to reduce the chances of over/underflow. In practice, it tends to be a little more stable to train for models with > 20bn parameters.

Mixed precision training means we have the weights, activations and optimizer states in different precision. Typically, your optimizer states need to be in fp32, activations are fp16 and the weights can be in either, depending on the set up.

Since blfoat16 has the same dynamic range as fp32, we don't need to use gradient scaling.

There are two different approaches to mixed precision:

![](_attachments/Screenshot%202022-07-13%20at%2016.46.10.png)

O(1) is PyTorch's default mixed precision. O(2) might have to be implemented ourselves?

The key point is that O2 is the same as O1, except it puts parameters in half precision. Gradient buffers have to be maintained in fp32 because doing many summations in bf16 can result in errors due to its low precision. The reason we need many summations is that we may be training with hundreds or thousands of GPUs --> we have to sum gradients across all of these after every backward pass.

## Accelerating Layernorm, Dropout, and Activation Functions
We can fuse element-wise into a single *kernel call*. For instance, both the $\textrm{sum}(x)$ and $GeLU()$ operations below are element-wise:

![](_attachments/Screenshot%202022-07-13%20at%2016.53.00.png)

For this, we can use Torch's JIT fusion. We would typically write the forward pass as normal, and decorate it with `@torch.jit.script`:

![](_attachments/Screenshot%202022-07-13%20at%2016.55.29.png)

We would also have to do the same thing with the backwards pass:

![](_attachments/Screenshot%202022-07-13%20at%2016.55.54.png)

Some examples of fused operations we can use are optimizers, layernorm and softmax:

![](_attachments/Screenshot%202022-07-13%20at%2016.57.39.png)

## Parallelism Nomenclature
It might be useful to understand properly the terms associated with training models across multiple devices.

![](_attachments/Screenshot%202022-07-13%20at%2017.00.59.png)

A **local rank** is an index of each GPU that is "local" to a particular node. **Global ranks** are global across multiple GPUs:

![](_attachments/Screenshot%202022-07-13%20at%2017.02.06.png)

**Data parallel ranks** index GPUs according to the data they recieve. For instance, in the case without model parallelism, each GPU recieves a different data input and so has a different DP rank:

![](_attachments/Screenshot%202022-07-13%20at%2017.05.01.png)

We can think of model parallelism as requiring **virtual GPUs**. Each copy of your model is contained on a single virtual GPU. We assign each GPU within a single virtual GPU with a Tensor Parallel rank:

![](_attachments/Screenshot%202022-07-13%20at%2017.06.47.png)

In this case, the data parallel ranks are the same within a virtual GPU:

![](_attachments/Screenshot%202022-07-13%20at%2017.08.40.png)

## Model and Data Parallelism

### Distributed Data Parallelism
Very familiar concept. Your model is copied across multiple GPUs, and each GPU receives different data:

![](_attachments/Screenshot%202022-07-13%20at%2017.11.58.png)

One interesting point is that as soon as the backward pass has been completed for a layer, an **allreduce** is *immediately* performed. This can save a lot of time:

![](_attachments/Screenshot%202022-07-13%20at%2017.13.55.png)

This is an all-to-all communication. 

### Tensor Parallelism
For more info, I've made some [more detailed notes](../ML/Tensor%20Parallel.md) on this topic.

This is one kind of **model parallelism**. The idea is that it spreads each weight matrix across multiple GPUs. It is intra-layer parallelism; you still have all of the layers of your network on a single GPU, but you only have a *part* of each layer on each device.

Let's take the example of an MLP. Here, we have two weight matrices, $W$ and  $V$. We split these by columns and rows respectively. During the forward pass, we send the input vector, $X$, to every GPU. 

![](_attachments/Screenshot%202022-07-15%20at%2011.11.24.png)

An important point here is that the matrices $W$ and  $V$ are divided such that the computations done by $XW_1$ only need to be multiplied by $V_1^T$, etc. In other words, $V_{2,3,4}^T$ don't need to care about the output for $XW_1$. If we think about this matrix multiply, then it's clear that we need to *sum* the outputs from each GPU at the end of the forward pass. This is the reason for the allreduce. 

There is a lot of communication overhead here: we must scatter $X$ across GPUs, and needs to be gathered at the end of the forward pass. This is before we even consider the backwards pass.

### Pipeline Parallelism
Pipeline parallelism involves putting entire layers onto a single GPU. It inter-layer parallelism. This is probably a bit easier to understand than tensor parallelism.

![](_attachments/Screenshot%202022-07-15%20at%2011.22.34.png)

We can think of this as a conveyor belt, where an input is passed from one GPU to the next. It's clear how we can parallelise this during the forward pass:

![](_attachments/Screenshot%202022-07-15%20at%2011.29.00.png)

Once the forward pass is done, we get a stack that starts to build up (see GPUs 2 and 3):

![](_attachments/Screenshot%202022-07-15%20at%2011.30.03.png)
![](_attachments/Screenshot%202022-07-15%20at%2011.34.33.png)

Once the whole backwards pass has completed, we can compute the Data-Parallel gradient all-reduce and do the SGD update.

A big problem with this approach is that some of the GPUs are sitting idle during the pipeline. There are many different pipeline parallel implementations. One is called GPipe, and is a little more simple than what we just saw (it waits for all of the forward passes to finish before doing the backward pass):

![](_attachments/Screenshot%202022-07-15%20at%2011.38.11.png)

The one we saw was a little more complex, where the forward and backward passes are interspersed:

![](_attachments/Screenshot%202022-07-15%20at%2011.38.43.png)

One way to alleviate how many GPUs are sitting idle during the pipeline is to **interleave** the layers that sit on each GPU. We don't have to be strictly contiguous. For instance, GPU 0 could have layers 0-4 as well as layers 12-17 (or something like that). This means that we end up looping back to a previous part of the pipeline at certain points:

![](_attachments/Screenshot%202022-07-15%20at%2011.41.43.png)

The details don't matter too much here, but the overall idea is interesting!

The main reason to use pipeline parallelism is if we max out our tensor parallelism on a particular node. Tensor parallelism also scales poorly across nodes, as it is so communication heavy. Pipeline parallelism is less communcation-heavy, so is better in settings where we have slow interconnect, or you must parallise you model across multiple nodes.

### Tensor + Pipeline Parallelism
We can combine tensor and pipeline parallelism. Consider a network with 36 layers:

![](_attachments/Screenshot%202022-07-15%20at%2011.47.14.png)

Now imagine splitting this horizontally. We get pipeline parallelism:

![](_attachments/Screenshot%202022-07-15%20at%2011.49.28.png)

Splitting vertically means we now get tensor parallelism too:

![](_attachments/Screenshot%202022-07-15%20at%2011.50.00.png)

Each pipeline stage is now spread across two GPUs. 

#### Tensor vs Pipeline Parallelism (Trade-Offs)
* Tensor parallelism involves a lot of frequent scatter/gather communication between GPUs. This typically necessitates very fast interconnect like NVlink.
* Tensor parallelism likes scaling a model's width instead of depth. If we were to scale purely along depth, and increase our tensor parallel size, then each of the matrix multiplies performed on one GPU would get smaller. This means the GPU wouldn't be kept "busy".

* Pipeline parallelism involves less communication, but having some idle GPUs is very hard to overcome.
* Pipeline parallelism is really useful when training very deep models.

* When using **3D Parallelism (Data + Tensor + Pipeline)**, parallelism is configured with Tensor Parallelism within a node and Pipeline Parallelism across nodes.

## Sequence Parallelism of Layernorm and Dropout Layers
As model size increases, the relative amount of memory associated with activations increases. The following bar chart was produced for GPT-3 style models (only focus on the "baseline") for now.

![](_attachments/Screenshot%202022-07-15%20at%2012.01.46.png)

Why is this? Because when using tensor parallelism, layernorm and dropout aren't split across GPUs. They are replicated on each device, because they cannot be parallelised along the hidden dimension.

## Selective Activation Recomputation


