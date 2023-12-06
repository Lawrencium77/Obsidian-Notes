These notes are based on [this](https://zdevito.github.io/2022/08/04/cuda-caching-allocator.html) Zach Devito blog. The document gives an overview with pseudocode for how the PyTorch CUDA Caching allocator behaves.

For reference, the CUDA caching allocator source code can be found [here](https://github.com/pytorch/pytorch/blob/a993319a4b34410ab0c655b44f7c98770d2f6ebc/c10/cuda/CUDACachingAllocator.cpp#L4). I haven't read it in detail, but it's useful to skim through.

A general tip: several code blocks reference functions which are only explained later on in the doc. Be patient!

```toc
```

## Introduction
> [!INFO]
> A bit of background: `cudaMalloc`, `cudaFree`, and `cudaMemcpy` are functions contained in the CUDA programming model which perform allocation, deallocation, and host-device data transfer.

The goal of the CUDA caching allocator in PyTorch is to reach a steady state where the program runs without needing to request new memory from CUDA using `cudaMalloc`. 

PyTorch relies on the CPU execution running ahead of GPU execution to hide the latency of the Python interpreter behind more time-consuming CUDA operations. But these memory APIs, especially `cudaFree`, introduce synchronisation which interferes with this process. We must therefore try to avoid CUDA memory API calls.

The caching allocator requests blocks of memory from CUDA and figures out ways to split up and reuse these blocks without returning them to CUDA.

Why not just request all GPU memory and manage it inside PyTorch? The answer is that PyTorch isn't the only library to use the CUDA memory APIs. cuBlas, cuDNN, and NCCL all do some allocations on their own, and users might mix PyTorch with other CUDA-accelerated libraries that we do not know about.

Instead, if PyTorch ever uses $N$ bytes of memory at one point, we'll continue to keep those $N$ bytes cached until a user specifically frees it via `torch.cuda.empty_cache()`.

Under normal circumstances, the allocator achieves these goals and mostly lives in the background. But sometimes a particular set of adversarial allocations might prevent it from reaching a steady state. In this case, it's useful to understand more about what the allocator is doing.

## Allocation
The overall approach to allocating memory is pretty simple. We maintain a cache of allocated blocks that we've previously gotten from CUDA, and attempt to reuse them:

Each block can be represented by the following struct:

```cpp
struct Block {
	void* addr;
	size_t size;
	bool active;
	// "set" in C++ is a sorted collection of unique objects
	set<Block>* pool; 
	// for splitting and merging blocks
	Block* prev, next;
};

// Create two empty sets
set<Block> small_pool;
set<Block> large_pool;
```

For "small" tensors (< 1MB), we use a separate pool to avoid fragmenting the larger pool. At a high level, we try to find a free block from this pool, but ask CUDA for more memory otherwise:

```cpp
Block malloc(int device, size_t size, cudaStream_t stream) {
	process_cross_stream_delayed_free()
	size = round_size(size);
	pool = size < 1MB ? small_pool : large_pool;

	Block block = <find and remove the smallest block in the pool on the same stream that is big enough to fit size>
	if (<block not found>) {
		block = alloc_new_block(size);
		block.pool = pool
	}
	block = maybe_split_block(pool, size, block)
	return block;
}
```

> [!INFO]
> Just to clarify, this stuff inside <> can represent a high-level description of some process or condition.

The allocation starts with some cleanup ([Streams](#Streams%20and%20Freeing%20Memory)) and [Allocation Rounding](#Allocation%20Rounding) of the size to be allocated (see below). It then uses a [best fit](Dynamic%20Memory%20Allocation#^7417a3) strategy finding the smallest block that fits for the allocation. If none can be found, then we ask CUDA for more memory.

Finally, the block of allocated memory we choose may be significantly larger than what was requested. This can happen if we are reusing a previous bigger allocation for a smaller one. In this case, `maybe_split_block` can split the block so the rest can be used for other allocations.

Blocks are allocated **per CUDA stream**. Each CUDA stream will have its own cache of allocations, with rebalancing done only by emptying all the caches in low_memory conditions.

## Allocation Rounding
Each allocation is rounded up to make it less likely to cause [fragmentation](Dynamic%20Memory%20Allocation#Fragmentation), as a result of oddly-shaped allocations. Rounding happens twice. First on the size being requested, and again if we are requesting a (possibly bigger) block from CUDA. 

The first of these (rounding of requested size) happens using `round_size`:

```cpp
size_t round_size(size_t size) {
	if (<default>) {
		return <round to a multiple of 512B>
	} else { // via configuration using PYTORCH_CUDA_ALLOC_CONF
	<round to a power of 2 with N divisions between, minimum size is 512>
      // For example, if we need to allocate 1200 and number of divisions is 4, the size 1200 lies between 1024 and 2048 and if we do 4 divisions between them, the values are 1024, 1280, 1536, and 1792. So the function will return 1280 as the nearest ceiling of power-2 divison.
	}
}
```

The environment variable `CUDA_PYTORCH_CUDA_ALLOC_CONF=roundup_power2_divisions:N`  (see [here](https://pytorch.org/docs/stable/notes/cuda.html#environment-variables)) makes the rounding more aggressive[^fn2]. This is done to avoid situations where small changes in batch size or sequence length will cause different-size allocations each time, making it harder to reach a steady state. Some memory is lost to this rounding; $N=1$ will on average waste 1/4th of each allocation[^fn1] but $N=2$ will only waste 1/8th. Since many models do not use varying sizes, it is disabled by default.

The second type of rounding is described in the [next section](#Asking%20CUDA%20For%20More%20Memory).

## Asking CUDA For More Memory
A second set of rounding rules applies when we need more memory from CUDA. This rounding happens as follows:

```cpp
Block alloc_new_block(size_t size) {
	// we always make a larger allocation than is requested.
	// this allows us to reuse the block, by spltting it.
	if (size < 1MB){
		allocation_size = 2MB;
	} else if (size < 10MB) {
		allocation_size = 20MB
	} else {
		allocation_size = <size rounded to a multiple of 2MB>
	}
	memory = cudaMalloc(allocation_size)
	if (<cuda is out of memory>) {
		return_our_possibly_fragmented_memory_to_cuda();
		memory = cudaMalloc(allocation_size);
		if (<cuda still out of memory>) {
			<throw an out of memory exception>
		}
	}
}
```

We see that these rules are pretty simple. We can also see that if we hit an OOM, the first step is to try and free up cached by unused memory (`return_our_possibly_fragmented_memory_to_cuda`). Before we see how this works, we need to see how `Block`s are split and recombined.

## Block Splitting/Merging
We ask CUDA for larger blocks of memory than we need for small (< 1MB) and medium size (1 - 10MB) allocations, with the intention of using this block for multiple allocations:

```cpp
Block maybe_split_block(Pool pool, size_t size, Block block) {
	remaining = block.size - size
	// Small pool -> split if remainder is > 512B
	// Large pool -> split if remainder is >= 1MB
	should_split = (size < 1MB && remaining > 512B) || (size >= 1MB && remaining > 1MB)
	if (!should_split) {
		return block;
	}
	block, rest = <split the first 'size' bytes into in new block, leaving a remaining block>
	pool.add(rest);
	return block;
}
```

The remaining block is added to the pool and can be used for another allocation. Here is a graphic of this process:

![](_attachments/Screenshot%202023-07-01%20at%2014.09.35.png)

Note that in the [actual PyTorch code](https://github.com/pytorch/pytorch/blob/a993319a4b34410ab0c655b44f7c98770d2f6ebc/c10/cuda/CUDACachingAllocator.cpp#L61), another condition is considered when calculating `should_split`; if `size >= 200 MB`, then the block cannot be split. This is also with the aim of reducing memory fragmentation.

Eventually, a `free` will return the block to the cached pool. To avoid fragmentation of the big slab of memory, when a block is returned, it will merge with its neighbours if they're also free:

```cpp
void return_block_to_reserved_memory(Block block) {
	if (<block was split from larger block>) {
		if (<successor of predecessor of split block is free>) {
			<merge with free successor and predecessor>
		} else {
			pool.add(block)
		}
	} else {
		pool.add(block)
	}
}
```





## Streams and Freeing Memory
We treat memory allocation and freeing as events that happen in order on a stream, just like CUDA kernels. This lets us schedule a kernel using an allocation  even if the kernel currently using that allocation hasn't finished.

However, there are no ordering guarantees between streams, so extra care must happen when a tensor `A` is allocated on one stream, `creator`, but used on another stream, `consumer`. In these cases, users have to call `A.record_stream(consumer)` to let the allocator know about `A`'s use on `user`. 

During `free`, the allocator will only consider the block ready for reuse when all work that has been scheduled up to the point `A` became free on `user` is complete. This is done by recording a CUDA event on the `user` stream and only handing out `A`'s member after that event has elapsed from the perspective of the CPU:

```cpp
void free(Block* block) {
	if (<block A has been used on a stream 'user' that it was not 'created' on>)
	 <record a cuda event to the end of 'user' stream and save it with A>
	 <Defer freeing A until those events have elapsed>
	 // condition is checked in process_cross_stream_delayed_free()
	 } else {
		 return_block_to_reserved_memory(block)
	 }
}

// called right before malloc
void process_cross_stream_delayed_free() {
	for all blocks waiting on events that have no remaining events {
		return_block_to_reserved_memory(block)
	}
}
```

Notice that even when a block is used on a different stream, its memory is always returned to the stream where it was allocated.

> [!QUESTION]
> I kind of get the idea here, but not totally. 
> I don't get exactly how the CUDA event part of this code works. Side note: see [here](https://chat.openai.com/share/277509c0-83a2-4235-831a-faf2c609c367) for an example of `tensor.record_stream()`.


## Low Memory Conditions and cudaMalloc retry
The final piece of the allocator is how to recover during an OOM. This is referred to as `cudaMalloc` retry in the statistics. When we have a lot of cached blocks, it is possible that a lot of them are free but too small to fulfil a large allocation - in other words, we have [fragmentation](Dynamic%20Memory%20Allocation#Fragmentation). This can be especially true if the allocation is a larger version (e.g. larger batch, longer sequence) of allocations from a previous step.

The caching allocator cannot move these fragmented blocks around to construct a larger allocation. But CUDA can move this memory around by changing page tables on the GPU. So the caching allocator's approach is to return these blocks to CUDA via `cudaFree` and try again:

```cpp
void return_our_possibly_fragmented_memory_to_cuda() {
	<wait for all cross stream events to complete>
	<for all free blocks in our reserved memory, cudaFree them>
}
```

In essence, CUDA rearranges memory to allow for large allocations. Here is a diagram to help explain:

![](_attachments/Screenshot%202023-07-01%20at%2014.34.45.png)

Since `cudaFree` synchronizes the device, this process is inefficient. So we use this time to also free any of the cross stream blocks we were waiting for as well.



At this point if we are out of memory, we raise an `OutOfMemoryError` exception. Under some circumstances, it might be the case that there is enough memory in the system to continue but we cannot free it. For instance, if there is a small tensor allocated in a big block of memory, we cannot free that block, wasting the block.

Something to keep in mind upon hitting an OOM is that the model is probably right in the middle of a step, near the end of the forward pass (since we are keeping a lot of temporary data for the backward pass). It's possible that if you were to back up to the beginning of a step then empty the allocator caches might be more successful. This means it might succeed at the next iteration, since it will directly `cudaMalloc` tensors of the new (larger) size:

```python
retry = False
try:
	step()
except torch.cuda.OutOfMemoryError:
	retry = True
	# exception handlers hold onto the stack trace frames, so 
	# the memory of temporary data will still be alive untik
	# we exit the exception handler
if retry:
	torch.cuda.memory.empty_cache()
	step() # might succeed now
```

To be clear, `torch.cuda.empty_cache()` releases all *unoccupied* blocks to the GPU. 

> [!QUESTION]
> Are `return_our_possibly_fragmented_memory_to_cuda()` and `torch.cuda.memory.empty_cache()` basically doing the same thing?

## The Meaning of Metrics
The allocator provides a number of metrics, with functions like `torch.cuda.memory_stats()`. These correspond to the ideas described above:

* `allocated_bytes` - The sum of the size of all Blocks in an active state. This includes memory added by `round_size`. This does not include blocks waiting to be freed in `process_cross_stream_delayed_free`.
* `reserved_bytes` -  The sum of the size of all blocks, regardless of state.
* `inactive_split_bytes` - The sum of the size of all Blocks in an inactive state that were split from a larger segment via `maybe_split_block`. This doesn't include blocks waiting to be freed in `process_cross_stream_delayed_free`. These are potential sources of fragmentation because they cannot be returned to CUDA during a `cudaMalloc` retry since another part of the block is still occupied.
* `cudaMalloc retries` - the number of times `return_our_possibly_fragmented_memory_to_cuda()` has been called. If this is being called frequently, it suggests the allocator has failed to reach a steady state.
* `CUDA OOMs` - The number of times the allocator has thrown an `OutofMemoryError`. Most non-interactive programs won't recover from this, but Python notebooks might accumulate multiple.


[^fn1]: $N=1$ means that the average allocation size is $2^N+2^N/2$. So the fraction used is $\frac{2^N+2^N/2}{2^{N+1}}=\frac{3\cdot2^{N-1}}{2^{N+1}}=\frac{3}{4}$. Similar logic applies to $N=2$.
[^fn2]: In other words, blocks are rounded up to larger sizes.