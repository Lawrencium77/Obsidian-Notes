These notes were largely based upon [this](https://leimao.github.io/blog/CUDA-Stream/#CUDA-Stream) Lei Mao blog.
```toc
```
## Basic Idea
In order to execute PyTorch code, there are three keys steps:

1. Host-to-device memory copy
2. CUDA kernel execution
3. Device-to-host memory copy

Intuitively, we would execute in this exact order. However, a serial approach might not be optimal. We can get performance gains by doing all three stages **concurrently**. This is where **CUDA streams** come into play.

## Execution Models
To explain CUDA streams, it is useful to have a model of how CUDA kernels are executed. We make the following assumptions:

* Memory copy time is a linear function of the size of memory to be copied
* GPU is never fully utilised
* Kernel execution can be divided into $N$ smaller kernel executions. Each smaller execution takes $1/N$ if the time of the original kernel execution
* Device-to-host, host-to-device, and kernel execution all take the same time
* Each CUDA engine executes kernels in order

We can invisage two models: a **serial** model and a **concurrent** model.

### Serial Model
![](_attachments/Screenshot%202022-09-13%20at%2013.36.45.png)

In the serial model, we execute linearly: host-to-device, kernel execution, and then device-to-host.

### Concurrent Model
![](_attachments/Screenshot%202022-09-13%20at%2013.37.49.png)

In the concurrent model, we execute all three stages asynchronously. In the above example, we use $N=4$ chunks. After completing host-to-device transfer of the first chunk, we launch kernel execution whilst simultaneously launching host-to-device transfer of the second chunk. And so on.

We can see that overlapping memory copy with computation allows us to significantly increase throughput.

The remaining question is how this is implemented in CUDA. The answer is with **CUDA streams**.

## CUDA Streams
According to the [CUDA programming guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#streams), a stream is a sequence of commands (possibly issued by different host threads) that execute in order. Different streams may execute their commands asynchronously w.r.t one another.

### Default Stream
According to the [CUDA programming guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#streams), kernel launches, host-to-device memory copy, and device-to-host memory copy that don't specify a stream parameter are issued to the default stream.

It is also called the `null` stream, or stream 0.

The rest of this section of the blog seems to involve some CUDA technicalities.

### Example Models with CUDA Streams
This figure illustrates how CUDA streams implement concurrency:

![](_attachments/Screenshot%202022-09-13%20at%2013.52.17.png)

> [!INFO]
> To be clear, these multiple CUDA streams are executing on the same device. 

### Kernel Execution Concurrency
In theory, kernel executions on different CUDA streams could run simultaneously (provided computational resources are sufficient). The example above assumes that times for host-to-device transfer, kernel execution, and device-to-host transfer are equal.
In practice, kernel executions on different CUDA streams are not exclusive; they could have some overlap.



