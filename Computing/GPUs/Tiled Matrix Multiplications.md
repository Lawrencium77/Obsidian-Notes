These notes are based on [this blog](https://penny-xu.github.io/blog/tiled-matrix-multiplication) by Penny Xu.
```toc
```

## Intro 
Tiling can be used to reduce global memory accesses by taking advantage of shared memory on the GPU. It boosts execution efficiency of a kernel.

The original post was intended to give an intuition on which each thread is doing in a basic tiled matmul.

## Some Background
In CUDA, threads are organized as follows:

* **Threads**: a single unit of execution - each thread has its own register memory;
* **Blocks**: groups of threads - all threads in a block has access to *shared memory*. A reminder that each streaming multiprocessor has its own shared memory.
* **Grid**: groups of blocks - all threads in a grid has access to *global memory*.

This is also covered in my notes on a [Summary of the CUDA Programming Model](Summary%20of%20the%20CUDA%20Programming%20Model.md).

## Problem Setup
Given a $4\times4$ input matrices A and B, we want to calculate the $4\times4$ output matrix C. Since C consists of 16 elements, let's launch 16 threads, where each thread calculate 1 output element.

The threads are organized into $2\times2$ blocks. There are 4 blocks in a grid.

![](_attachments/Screenshot%202022-09-29%20at%2015.12.32.png)

## Visualisation
Let's see what each thread is doing.

Each thread is responsible for loading input elements into shared memory. Since this is shared within a block, it means that each of the four threads in a block can access the data loaded by the other three threads.

On a high level, the process is to iteratively calculate partial sums for the output block. We keep a running total of these, and the final number is written to our output matrix.

I found the order in which threads load data to be a little confusing at first, but it does make sense!

To begin, each thread loads an element from a block of matrix A.

![](_attachments/Screenshot%202022-09-29%20at%2015.15.02.png)

And loads them into shared memory:

![](_attachments/Screenshot%202022-09-29%20at%2015.15.44.png)

Similar then happens for matrix B:

![](_attachments/Screenshot%202022-09-29%20at%2015.16.23.png)

We then compute the temporary result for the matmul of these two blocks, and store them.

![](_attachments/Screenshot%202022-09-29%20at%2015.17.08.png)

This is repeated for the second blocks of matrices A and B, to get our final result! 

![](_attachments/Screenshot%202022-09-29%20at%2015.18.40.png)

### Benefit of Tiling
**Without tiling**: in order to calculate one output element, a thread needs to access an entire row of A, and an entire column of B. This means there are **8 accesses per thread**.

**With tiling**: Each thread ends up loading two elements from A, and two elements from B. This means there are **4 accesses per thread**.

In general, reduction in global memory accesses in tiled matmuls is proprtional to the dimension of the blocks used. This means that with blocks of size $N\times N$, the potential reduction of global memory traffic would be $N$. 

## Kernel Code
I don't fully understand kernel code, as it's written in CUDA (and I don't have a full knowledge of this). However, it's still useful to look at to get an idea of what's going on.

A few points:

* The argument `width` is the width of output C;
* The number "2" is used a lot in the code, since it matches our tile width;
* `threadIdx` is specific to a block, and `blockIdx` is specific to a grid. Since our matmul has a 2D output, it's easiest to organize the threads in 2D. So the four threads in a block are actually organized like thread00, thread01, etc. This is also the case for how each block (block00, block01, etc.) is indexed in this example.
* `__syncthreads` is a barrier synchronisation line that says no threads can continue execution of the remaining code until all threads have reached that point in their execution. This is super important for the correctness of the algorithm.

The code is as follows:

```C
__global__ tile_matrix_multiply(float* A, float* B, float* C, int width)

    __shared__ shareA[2][2]; // Define some shared memory 
    __shared__ shareB[2][2];
    int bx = blockIdx.x; int by = blockIdx.y; // blockIdx is a struct?
    int tx = threadIdx.x; int ty = threadIdx.y;
    int row = by * 2 + ty; //
    int col = bx * 2 + tx;
	float temp = 0;   
	for(int i = 0; i < width/2; ++i){

		shareA[ty][tx] = A[row*width + (i*2 + tx)];
		shareB[ty][tx] = B[(i*2 + ty)*width + col];
		__syncthreads();

		for(int k = 0; k < 2; ++k){
			temp += shareA[ty][k] * shareB[k][tx];
			__syncthreads();
	    }
	}
	C[row*width + col] = temp;
```
