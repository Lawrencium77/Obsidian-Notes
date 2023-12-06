I've split this doc into three parts. The first two provide context, and the third describes my confusion.

```toc
```

## Allocator Requirements and Goals
* The designer of a memory allocator aims to meet often conflicting performance goals:

> 1. **Maximising throughput** - the rate at which an allocator completes requests.
> 
> 3. **Maximising memory utilisation** - It's incorrect to assume that virtual memory is an unlimited resource. Instead, the amount of VM allocated by all the processes in a system is limited by the amount of [swap space](Memory%20Mapping#^05c2fd) on disk. 
> 	There are several ways to characterise how efficiently an allocator uses the heap. According to the CSAPP authors, the most useful metric is **peak utilisation**. Suppose we're given a sequence of $n$ allocate and free requests:
$$ R_0, R_1, \dots, R_{n-1} $$
If an application requests a block of $p$ bytes, the resulting allocated block has a **payload** of $p$ bytes. After request $R_k$ is complete, let the **aggregate payload**, $P_k$, be the sum of payloads of the currently allocated blocks. Let $H_k$ denote the current (monotonically nondecreasing) size of the heap.
Then the peak utilisation of the first $k+1$ requests is:
$$U_k=\frac{\max_{i \le k}P_i}{H_k}$$

The objective of the allocator is to maximise $U_{n-1}$ - the peak utilisation over the entire sequence.

## Implementation 
* The simplest imaginable allocator would organise the heap as a large array of bytes and a pointer `p` that initially points to the first byte of the array.
* To allocate `size` bytes, `malloc` would save the current value of `p` on the [the stack](Educative%20Course.md#The%20Stack), increment `p` by `size`, and return the old value of `p` to the caller.
* This naive allocator is an extreme point in the design space. Throughput would be extremely good, since each `malloc` and `free` execute only a handful of instructions.
* However, since the allocator never reuses any blocks, memory utilisation would be extremely bad.

## My Question

I don't understand why memory utilisation is important at all. 

My read is that the limit on VM is that the sum of **allocated** memory across all processes cannot exceed the amount of physical memory available (DRAM + Swap Space).

In other words, heap size is not what matters, but instead the amount of allocated memory. 

However, their metric of "memory utilisation" suggests that the amount of free memory does matter. Furthermore, their "simple" allocator (which just increases the heap size whenever there is a `malloc`) seems fine to me - but obviously the authors disagree.

What's going on? Why does external memory fragmentation matter?
