# Graphical Processing Units (GPUs)
## Motivation: Matrix Multiplication
Consider the multiplication of 2 matrices $K=MN$. The mathematical operation to compute one element of $K$ is essentially a dot product: $x=ad+be+cf+...$. Technically, this is called **Fused Multiply Add (FMA)**. It requires $2N-1$ mathematical operations, where $N$ is the number of terms to be added. ^77feae

If our matrix shapes are $(a\times c)=(a\times b)(b\times c)$ then the total matmul requires $O(2abc)$ mathematical operations.

A **Central Processing Unit (CPU)** would do each of these $2abc$ operations sequentially. However, we could instead **parallelise** the process since, in general, the elements of $K$ can be computed independently of each other (this changes if $M$ or $N$ have some special structure, e.g. block diagonal).

## Structure
A **core** is the component of a processor that executes an instruction. 

In a CPU, each core has associated with it a certain number of registers. The core executes functions on these registers. A 32-bit core means that each register possesses 32 bits of memory. The **clock rate** of a processor is related to the rate at which the processor can execute these functions.

The memory hierarchy of a GPU is shown below:

![](_attachments/Screenshot%202022-03-03%20at%2013.31.57.png)

A key component of a GPU is the **streaming multiprocessor (SM)**. 

Each SM contains multiple cores. Each core has associated registers, which only it can access. There also exists L1 cache aka **shared memory**, which is on-chip and can be accessed by all cores in the SM.
Each SM also contains **read-only memory**. This includes **instruction cache**, constant memory, texture memory and RO cache. 

Each GPU has L2 cache, that is accessible to all SMs. Finally, there also exists **GPU DRAM** which, again, can be accessed by all SMs on the GPU.

For reference, an NVIDIA A100 has:
* 108 SMs;
* In total, >7000 cores;
* ~70 cores per SM.

## Threads, Thread Blocks and Kernels
![](_attachments/Screenshot%202022-03-03%20at%2013.37.56.png)

GPUs are very good at **SIMD: Single Instruction on Multiple Data**.
An example instruction is FMA; an example of multiple data is multiple matrix elements.

Each SM is assigned a single instruction.

Each sequence of instructions on a GPU is executed by a [thread](The%20Operating%20System%20Manages%20the%20Hardware#Threads). Each thread is executed by a different core. We can think of a thread as a series of actions, that does single instruction on a single data.

In the context of matrix multiplication, a single thread would be a single FMA.

We can combine threads into a **thread block**. These are executed by an SM. Each thread block cannot be migrated to other SMs in the GPU, but one SM can run several concurrent thread blocks depending on the resources required.

We can combine thread blocks into a **kernel grid**. These are executed by a GPU. It is possible to run multiple kernels on a device at one time.

For reference, a **kernel** is the instruction that is executed by the GPU. The key point about kernels is that the kernel doesn't change, but the *data* does. In this sense, it is similar to a function.

## Tiling
In the context of matrix multiplication, the output matrix $K$ is split into a series of **tiles**. Each tile is sent to an SM. If the tiles are small enough, then all input data can be put into shared memory. This makes the resulting computation extremely quick.

Lastly, **memory bandwidth** refers to how quickly we can load from DRAM onto chip. It is usually expressed in bytes/second.

### GPUs vs CPUs
![](_attachments/Screenshot%202022-03-03%20at%2014.00.29.png)

GPUs dedicate most of their transistors - the key active component in electronic systems - for data processing. [CPUs](../Computational%20Logic%20and%20CPUs.md) also need to reserve [die area](https://en.wikipedia.org/wiki/Die_(integrated_circuit)) for big caches, [control units](https://en.wikipedia.org/wiki/Control_unit) and so on.

CPUs aim to minimize latency within each thread, while GPUs hide the instruction and memory latencies with computation. This idea is shown below:

![](_attachments/Screenshot%202022-03-03%20at%2014.07.49.png)

CPUs achieve this low latency with large caches and complex control logic. Caches work best with only a few threads per core, as context switching between threads is expensive. CPUs can do multithreaded programming (but not to the same extent as high throughput accelerators). Their key benefit is that they can do a wide range of arithmetic and memory operations.

GPUs have a smaller range of ops available. This means they have fewer transistors & gates, so are easily parralelisable. The motivation behind this is that to produce graphics, a lot of matrix multiplication is required. Addition and multiplication are simple operations, so require few transistors. This means little space is needed on the chip. 

In GPUs, threads are lightweight, so a GPU can switch from stalled threads to other threads very easily. The figure above shows that when thread T1 stalled for data, another thread T2 started processing, and so on with  T3 and T4.  In the meantime, T1 eventually gets the data to process. 

In this way, latency is hiding by switching to other, available work. This means that GPUs need many overlapping concurrent threads to hide latency. As a result, you want to run thousands of threads on GPUs.