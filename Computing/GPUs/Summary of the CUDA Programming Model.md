These notes are based on [this](https://developer.nvidia.com/blog/cuda-refresher-cuda-programming-model/) Nvidia blog.

## Intro
CUDA provides an abstraction of GPU architecture that acts as a bridge between an application and its possible implementation on GPU hardware. These notes outline the main concepts of the CUDA programming model.

We shall begin by introducing two keywords widely used in CUDA: **host** and **device**.

The **host** is the CPU available in the system. The system memory associated with the CPU is called host memory. The GPU is called a **device** and GPU memory is called device memory.

To execute any CUDA program, there are three main steps:

1. Copy the input data from host memory to device memory. Also known as **host-to-device transfer**
2. Load the GPU program and execute, caching data on-chip for performance
3. Copy the results from device to host memory, also called **device-to-host transfer**.

Device-host information transfer happens over the PCIe bus.

## CUDA Kernel and Thread Hierarchy
The following figure shows that **a CUDA kernel is a function that gets executed on GPU**. The parallel portion of your applications is executed $K$ times in parallel by $K$ different CUDA threads. This is instead of regular C/C++ functions, which are executed only once.

![](_attachments/Screenshot%202022-08-09%20at%2016.59.41.png)

A group of threads is called a **CUDA block**. CUDA blocks are grouped into a **grid**. A kernel is executed as a *grid of blocks of threads*:

![](_attachments/Screenshot%202022-08-09%20at%2017.00.56.png)

In other words, CUDA kernels are subdivided into blocks, which consist of threads.

Each CUDA block is executed by one streaming multiprocessor (SM) and cannot be migrated to other SMs in GPU. However, one SM can run several concurrent CUDA blocks depending on the resources needed by CUDA blocks.

Likewise, each kernel is executed on one device (GPU) and CUDA supports running multiple kernels on a device at one time. However, an individual kernel cannot be migrated to a different device.

Figure 3 shows how the kernel execution maps to hardware resources available in GPU:

![](_attachments/Screenshot%202022-08-09%20at%2017.04.52.png)

## Summary
The CUDA programming model provides a heterogeneous environment where the host code is running the C/C++ program on the CPU, and the kernel runs on a physically separate GPU device. 

The CUDA programming model also assumes that both the host and the devi e maintain their own separate memory spaces - referred to as host memory and device memory respectively. CUDA code also provides for data transfer between host and device memory, over the PCIe bus.

