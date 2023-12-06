These notes were based on [this](https://lilianweng.github.io/posts/2021-09-25-train-large) blog by Lilian Weng. It relates very strongly to my notes on [Engineering Challenges in Training Massive Transformers](Engineering%20Challenges%20in%20Training%20Massive%20Transformers.md).

An individual GPU has limited memory. Large models have grown beyond a single GPU. There are several parallelism paradigms to enable model training across multiple GPUs, as well as a variety of model architecture and memory saving designs to help make it possible to train *very* large neural nets.

```toc
```


# Training Parallelism
The main bottleneck in training very large models is the intense demand for GPU memory. Besides the model weights, it is usually *even more* expensive to store intermediate computation outputs (such as the gradients), and optimizer states. 

As a result, we need parallelism. This can happen across different dimensions, including data, model archiecture, and tensor operation.

## Data Parallelism
The most naïve way to do **data parallelism (DP)** is to copy the same model weights into multiple workers and assign a fraction of data to each worker to be processed at the same time.

This can't work well if the model size is larger than a single GPU node's memory. 

At the end of each minibatch, workers need to synchronize gradients or weights. There are two main approaches to this, and both have clear pros/cons:

1. **Bulk synchronous parallel (BSP)**: Workers sync data at the end of every minibatch. It prevents model weights becoming stale, and has good learning efficiency. However, each machine has to halt and wait for others to send gradients.
2. **Asynchronous parallel (ASP)**: Every GPU worker processes data asynchronously - no waiting or stalling. However, this can lead to stale weights and thus lower the statistical learning efficiency.

Somewhere in the middle is to synchronize gradients globally once every $x$ iterations ($x>1$). This feature is called **gradient accumulation** in **Distributed Data Parallel (DDP)**. Multiple gradients are bucketed into one `AllReduce` to improve throughput. Computation and communication scheduling can be optimized based on the computational graph.[^fn1]

## Model Parallelism
**Model Parallelism (MP)** aims to solve the case when the model weights cannot fit into a single node. The computation and model parameters are partitioned across multiple machines. This means that both memory usage and computation are reduced.

Since deep neural nets usually contain a stack of vertical layers, it feels straightforward to split a large model by layer, where a small consecutive set of layers are grouped into one partition on one worker. However, a naïve implementation for running every data batch through multiple such workers with sequential dependency leads to big bubbles of waiting time and severe under-utilization of compute.

Here's a diagram of such a naive implementation:

![](_attachments/Screenshot%202022-08-04%20at%2009.04.09.png)


### Pipeline Parallelism

**Pipeline parallelism** combines model parallelism with data parallelism to reduce idle time of GPUs. The main idea is to split each minibatch into multiple **microbatches** and enable each worker to process one microbatch simultaneously. Note that every microbatch needs two passes: one forward and one backward. Inter-worker communication only transfers activations (forward) and gradients (backward). How these passes are scheduled and how the gradients are aggregated vary in different approaches. The number of partitions (i.e. workers) is also known as the **pipeline depth**.

In *GPipe*, gradients from multiple microbatches are aggregated and applied synchronously at the end. This guarantees learning consistency and efficiency, irrespective of the number of workers. Bubbles still exist, but are much smaller than the naïve case:

![](_attachments/Screenshot%202022-08-04%20at%2009.08.47.png)

Given $m$ evenly split microbatches and $d$ partitions, assuming both forward and backward passes for a microbatch take one unit of time, the fraction of bubble is[^fn2]:

![](_attachments/Screenshot%202022-08-04%20at%2009.10.31.png)

The GPipe paper observes that the bubble overhead is almost negligible if the number of microbatches is 4x the number of partitions, i.e. $m > 4d$. This relates to **activation recomputation** (see later).

*PipeDream* schedules each worker to alternatively process that forward and backward passes. There is a fair amount of detail to this, more of which is described in Lilian Weng's actual blog.

> [!INFO]
> I made some notes on how to do [Pipeline Parallel in PyTorch](PyTorch/Pipeline%20Parallel%20in%20PyTorch.md).

### Tensor Parallelism

>[!INFO]
I made some more [in-depth notes](../ML/Tensor%20Parallel.md) on this.

As an alternative to data parallelism, we can horizontally partition the computation for one tensor operation across multiple devices. This is **tensor parallelism**.

Let's take a transformer as an example. The transformer model mainly consists of layers of MLP and self-attention blocks. *Megatron-LM* adopts a simple way to parallelize intra-layer computation for MLP and self-attention.

An **MLP** layer in a transformer contains a general matrix multiply (GEMM) followed by a non-linear GeLU. Let's split weight matrix $A$ by column:

![](_attachments/Screenshot%202022-08-04%20at%2009.27.04.png)

The attention block runs GEMM with query $Q$, key $K$, and value weights $V$. For some input vector $X$, this can be written as:

![](_attachments/Screenshot%202022-08-04%20at%2009.29.10.png)

We can now illustrate how tensor parallelism operates for MLPs and attention layers:

![](_attachments/Screenshot%202022-08-04%20at%2016.07.01.png)

# Other Memory Savings
## CPU Offloading
When GPU memory is full, one option is to offload temporarily unused data to CPU and read them back when needed later. The idea of **CPU offloading** is straightforward but less popular in recent years due to the slowdown it adds to training.

## Activation Recomputation
**Activation recomputation** is otherwise known as **gradient checkpointing**. It's a smart yet simple idea to reduce memory footprint at the cost of computation. It reduces the memory cost of training a $n$-layer network to $O(\sqrt{n})$.

Let's say we evenly divide an $n$-layer network into $k$ partitions. Only activations at partition boundaries are saved. Other activations are still needed for computing gradients so they are recomputed during the backward pass.
With gradient checkpointing, the memory cost for training is:

![](_attachments/Screenshot%202022-08-04%20at%2016.27.12.png)

The first part of the equation is the memory cost to run backprop on each of the segments. 
The second is the cost of storing all the saved activations.
The total cost can be minimised by setting $k = \sqrt{n}$, giving a total cost of $O(\sqrt{n})$. The algorithm *only requires an additional forward pass* during training, but reduces the memory cost to be *sub-linear*. Since the backward operation is nearly twice as time-consuming as the forward pass, it slows down computation by a small relative amount.

> [!INFO]
> #### Applying this to Transformers
> Transformers are more complex than a series of FF layers! There are, of course, many different components. So it's not totally obvious how to apply Gradient Checkpointing.
> In practice, I think the solutions people use are pretty simple. At Speechmatics, we followed two approaches (at different points in time):
> * Save the output of every Transformer Block
> * Save the output of every MHA and FF block (i.e. twice for every Transformer Block).
> 
> The former gives slightly stronger memory savings than the latter.
> It would be good to get some confirmation on how other people do gradient checkpointing.


## Mixed Precision Training
When using mixed precision training, it is important to consider the trade-off between memory gains and accuracy. [This](https://arxiv.org/abs/1710.03740) paper describes some methods that can be used to prevent losing accuracy.

They describe three techniques:

1. **Full-precision master copy of weights**: Maintain an FP32 copy of model weights, which accumulate gradients (i.e. get changed according to the gradients at each step). These values are cast to half-precision for forward and backward passes. The motivation is that each gradient update might be too small to be fully contained within the FP16 range.

	This procedure is summarised in this figure:

![](_attachments/Screenshot%202022-08-04%20at%2017.14.43.png)
2. **Loss Scaling**: Scale up the loss to better handle gradients with small magnitudes. Scaling helps shift the gradients to occupy a larger section of the representable range of FP16, preserving values that are otherwise lost.
	
	In the blog, Lilian includes an interesting figure from the aforementioned paper. It shows a histogram of gradients during model training, in full precision.

![](_attachments/Screenshot%202022-08-04%20at%2017.19.26.png)
3. **Arithmetic Precision**: For common network arithmetic (e.g. dot products, reduction by summing vector elements), we can accumulate the partial results in FP32 and then save the final output as FP16 before saving into memory. Point-wise operations can be executed either in FP16 or FP32.

## Compression 
The output of intermediate layers consume a lot of memory, yet they are only needed in one forward pass and one backward pass. There is a noticeable temporal gap between these two uses. A **data encoding** strategy has been proposed to compress intermediate results after the first use in the first pass, and to then decode it for backprop later on.

The first paper to do this was by [Jain et al.](https://www.microsoft.com/en-us/research/uploads/prod/2018/04/fiddle-gist-isca18.pdf). A bit more detail is given in the blog.

## Memory Efficient Optimizer
Optimizers require considerable amounts of memory. Take Adam as an example: it needs to maintain both the first and second moments of the gradient.

Several optimizers have even proposed to reduce the memory footprint. For instance, instead of storing the full moments as in Adam, *Adafactor* only tracks the per-row and per-column sums of the moving averages and then estimates the second moments based on these sums.

**ZeRO (Zero Redundancy Optimizer)** optimizes the memory used for training large models based on an observation about two major areas of memory consumption of large model training:

1. The majorty of memory is occupied by *model states*, including optimizer states, gradients, and parameters. Mixed-precision training demands a lot of memory since the optimizer needs to keep an FP32 copy of parameters and other optimizer states.
2. The remainder of memory is consumed by activations, temporary buffers and unusable fragmented memory (named *residual states* in the paper).

ZeRO combines two approaches: *ZeRO-DP* and *ZeRO-R*.

Zero-DP is an enhanced version of data parallel. It partitions the optimizer state, gradients and parameters across multiple data parallel processes via a dynamic communication schedule to minimize the communication volume.
Zero-R optimizes the memory consumption of residual states, using partitioned activation recomputation, constant buffer size and on-the-fly memory defragmentation.

There is certainly a lot of detail here, so I don't really understand what's going on. It'll be best to read more about ZeRO, perhaps starting with the [original paper](https://arxiv.org/abs/1910.02054).






[^fn1]: I think this means that we can communicate gradients/buffers at specific points, that are related to the model architecture.
[^fn2]: The amount of processing time is $2md$. The total amount of time for the whole process to run is $(2(m+(d-1)))\times d$. Hence the form of this equation


