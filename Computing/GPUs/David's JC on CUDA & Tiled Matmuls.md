Simplest CUDA kernel (just adds two vectors together):

![](_attachments/Screenshot%202022-10-26%20at%2010.44.00.png)

The __global__ keyword defines the kernel. The blockIdx, blockDim and threadIdx variables are all accessible within the kernel. They are set within the bottom line somehow. The bottom line is the actual kernel launch.

In Triton Lang, we use ` @triton.jit` instead of the `__global__` method.

Why have blocks as a concept at all? It relates to the context of the execution unit. They all have access to the same shared memory, so can read/write from the same local memory. 

You can run multiple blocks on a single SM simultaneously. They read/write to the same shared memory, but **can only access their own partition of that memory.** The blocks that run in parallel on the same SM don't even have to be from the same CUDA kernel.

There is a **distinction between L1 cache and shared memory** - even though they use the same memory on an A100, they are used for different purposes! And the split between L1 and shared memory is configurable! 

Diagram of A100:

![](_attachments/Screenshot%202022-10-26%20at%2011.04.25.png)

Might be worth figuring out how David got this.

Tensor cores are special. They are specialised to do one thing: matmuls.

Indices:

![](_attachments/Screenshot%202022-10-26%20at%2011.07.15.png)

Warp is the actual unit of work that gets executed in a given clock cycle. They have size 32 (minimum), i.e. contain 32 threads. Each block is divided into warps. Each warp is scheduled to run against the hardware.

Each warp executes the same instruction.

The blocks in the diagram above are splits into two warps (each of size 4).

Suppose we are executing against a GPU with 4 SMs:

![](_attachments/Screenshot%202022-10-26%20at%2011.09.10.png)

A similar scenario, but with 1 warp per SMX:

![](_attachments/Screenshot%202022-10-26%20at%2011.10.32.png)

The main point here is that each block doesn't have to be executed in parallel.
Context switching:

![](_attachments/Screenshot%202022-10-26%20at%2011.11.08.png)

In this diagram, we can either alternate between warps, or execute them sequentially. But the performance is the same, because context switching is "free" in a GPU.

Context switching on CPU is expensive. Every time a thread switches context, it has to move its context from the register file to some other memory. On a GPU, context switching is designed to be quick. The context of each thread is always maintained on the register throughout its entire execution, so the context switch is free. 

Something about instruction pipeline being saturated. You always want to oversubscribe, such that the next thing is ready to run. The masks data movement cost with execution of work. 

Why do warps exist? Not sure

Branching logic can also screw things:

![](_attachments/Screenshot%202022-10-26%20at%2011.18.28.png)

Blocks 2 & 4 have no branching logic. But 1 & 3 do have branching logic. This is called **warp divergence**.

Some more cases are as below:

![](_attachments/Screenshot%202022-10-26%20at%2011.24.05.png)

Smaller tiles means less data reuse, but you are less likely to get bad tile quantization.

Summary from this part:

![](_attachments/Screenshot%202022-10-26%20at%2011.27.55.png)

#### Next Section

![](_attachments/Screenshot%202022-10-26%20at%2011.31.40.png)

Number of FLOPS here is O(2abc).

Arithmetic intensity - the amount of computation per memory access (roughly). An elementwise addition has low arithmetic intensity.

The bottleneck is obviously the data movement. 

In general, we are trying to increase our arithmetic intensity. This is what we are doing with fused kernels.

It's quite fun to think about the best- and worse- case arithmetic intensity here. Worst case: 128/128 = 1. Best case: 128/64 = 2.

Summary of compiler optimizations in CUDA vs Triton:

![](_attachments/Screenshot%202022-10-26%20at%2011.46.27.png)

We program at the block level instead of the thread level. This is easier to read about, and allows for automated optimizations at compile time (memory coalescing, shared memory management, scheduling within SMs).

What exactly is CUDNN? They've written kernels in CUDA.

Worth reading about **Flash Attention**. It's explained in Triton lang. The quadratic complexity of attention is an expensive. There are lots of cheaper attention approximations, but they don't seem to be widely used (according to David). Flash Attention gets speedups with hardware utilisation (instead of attention approximation):

1. It tries to go via the AOT Autograd route. It focuses on recomputation instead of reading/writing during the backwards pass. The $QK^T$ is computed incrementally instead of reading/writing it all at once? Or something like that. But this requires an incremental softmax (how does this work?)

Definitely worth reading this paper!

What is **hyper-threading**?

