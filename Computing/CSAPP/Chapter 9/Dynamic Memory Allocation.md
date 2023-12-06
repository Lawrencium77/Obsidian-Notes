```toc
```

## Intro Stuff
* While it is possible to use the low-level [mmap/munmap](Memory%20Mapping#User-Level%20Memory%20Mapping%20with%20the%20`mmap`%20Function) functions to create/delete areas of virtual memory, it's typically more convenient to use a [dynamic memory allocator](https://en.wikipedia.org/wiki/C_dynamic_memory_allocation) to acquire additional memory at runtime.

> [!INFO]
> To be clear, memory is allocated within a process's virtual address space.

* A dynamic memory allocator maintains an area of a process's virtual memory called the [heap](Learning%20C#Section%206:%20Memory%20-%20Stack%20vs%20Heap).
* We assume the heap is an area of [demand-zero](Memory%20Mapping#^b05890) memory that begins immediately after the uninitialised data area, and grows upward (towards higher addresses). ^de7871
* For each process, the kernel maintains a variable `brk` ("break") that points to the top of the heap:

![](_attachments/Screenshot%202023-05-09%20at%2016.47.25.png)

* An allocator maintains the heap as a collection of various-size **blocks**. 
* Each block is a contiguous chunk of virtual memory, that is either **allocated** or **free**.
* An allocated block has been explicitly reserved for use by an application.
* A free block is available to be allocated.
* An allocated block remains allocated until it is freed, either explicitly by the application or by the memory allocator itself.
* Allocators come in two basic styles. Both require the application to explicitly allocate blocks. They differ in the entity which is responsible for freeing blocks:

> 1. **Explicit Allocators:** Require the application to free any allocated blocks. For instance, the C standard library provides an explicit allocator called the `malloc` package. C programs allocate blocks using the `malloc` function, and free blocks by calling `free`.
> 2. **Implicit Allocators**: Require the allocator to detect when an allocated block can be freed. These are also called [garbage collectors](../../../Coding/Python/Garbage%20Collection.md). 

The remainder of this section discusses explicit allocators. [Section 9.10](Garbage%20Collection%20(CSAPP).md) discusses garbage collection.

## The `malloc` and `free` Functions
* The C standard library provides an explicit allocator called the `malloc` package.
* Programs allocate blocks from the heap by calling the `malloc` function:

```C
#include <stdlib.h>

void *malloc(size_t size);
```

* `malloc` returns a pointer to a block of memory at least size `size` bytes.
* It has to be suitably "aligned". For example, a 32-bit machine will return a block whose address is always a multiple of 8. I *think* this is necessary to ensure that blocks can hold any type of data object (?).
* Programs free heap blocks by calling `free`:

```C
#include <stdlib.h>

void free(void *ptr);
```

* `ptr` must point to the beginning of the allocated block that was obtained from `malloc`. 
* `free` doesn't return anything.
* Figure 9.34 shows how an implementation of `malloc` and `free` might manage a smaller heap of 16 words. Each box represents a 4-byte word. The heavy-lined rectangles correspond to allocated blocks (shaded) and free blocks (unshaded). Initially, the heap consists of a single 16-word free block. ^30899d

![](_attachments/Screenshot%202023-05-09%20at%2017.06.04.png)

Throughout this section, we assume the allocator returns blocks aligned to 8-byte double-word boundaries. Let's explain the above figure:

> * **9.34 (a)**: Program asks for a four-word block. `malloc` returns a four-word block from the front of the free block, and returns a pointer to the first word.
> * **9.34 (b)**: Program requests a 5-word block. `malloc` pads the block with an extra word in order to keep the free block aligned on a double-word boundary. It thus returns a 6-word block.
> * **9.34(c)**: Program requests a 6-word block and `malloc` responds.
> * **9.34(d)**: Program frees the 6-word block that was allocated in 9.34(b). After the call to `free` returns, the pointer `p2` still points to the freed block. It's the responsibility of the application not to use `p2` again until it's reinitialised by a new call to `malloc`.
> * **9.34(e)** The program requests a 2-word block. `malloc` allocates a portion of the block that was freed in the previous step and returns a pointer to this new block.

## Why Dynamic Memory Allocation?

* The alternative to dynamic memory allocation is **static memory allocation**. 
* Memory is allocated at compile time[^fn2], before a program runs.
* The most important reason that programs use dynamic memory allocation is that we often don't know the sizes of certain data structures until a program actually runs.
* For instance, suppose we need a C program that reads in a list of $n$ ASCII integers from `stdin` into a C array. The simplest approach is to define the array statically, with some hard-coded maximum array size:

```C
#include "csapp.h"
#define MAXN 15213

int array[MAXN];

int main()
{
	int i, n;
	scanf("%d", &n);
	if (n > MAXN)
		app_error("Input file too big");
	for (i = 0; i < n; i++)
		scanf("%d", &array[i]);
	exit(0);
}
```

* This is a bad idea, since `MAXN` is arbitrary and has no relation to the actual amount of virtual memory available.
* A better approach is to allocate the array dynamically at runtime:

```C
#include "csapp.h"

int main()
{
	int *array, i, n;
	
	scanf("%d", &n);
	array = (int *)Malloc(n * sizeof(int));
	for (i = 0; i < n; i++)
		scanf("%d", &array[i]);
	free(array);
	exit(0);
}
```

* With this approach, the maximum size of the array is limited only by the amount of virtual memory.

## Allocator Requirements and Goals
* Explicit allocators must operate within some pretty strong constraints.
* The main one being that it's able to handle **arbitrary request sequences** - an application can make an arbitrary sequence of allocate and free requests.

> [!INFO]
> I skipped a decent bit of detail on the constraints on memory allocators.

* Working within these constraints, the designer of an allocator aims to meet often conflicting performance goals:

> 1. **Maximising throughput** - the rate at which an allocator completes requests. It's not too hard to develop allocators with reasonably good performance where the worst-case running time of an allocate request is linear in the number of free blocks, and the running time of a free request is constant.


> 2. **Maximising memory utilisation** - It's incorrect to assume that virtual memory is an unlimited resource. Instead, the amount of VM allocated by all the processes in a system is limited by the amount of [swap space](Memory%20Mapping#^05c2fd) on disk. 
> 	There are several ways to characterise how efficiently an allocator uses the heap. According to the CSAPP authors, the most useful metric is **peak utilisation**. Suppose we're given a sequence of $n$ allocate and free requests:
$$ R_0, R_1, \dots, R_{n-1} $$
If an application requests a block of $p$ bytes, the resulting allocated block has a **payload** of $p$ bytes. After request $R_k$ is complete, let the **aggregate payload**, $P_k$, be the sum of payloads of the currently allocated blocks. Let $H_k$ denote the current (monotonically nondecreasing) size of the heap.
Then the peak utilisation of the first $k+1$ requests is:
$$U_k=\frac{\max_{i \le k}P_i}{H_k}$$

The objective of the allocator is to maximise $U_{n-1}$ - the peak utilisation over the entire sequence.
We'll see that there is a tension between maximising throughput and utilisation.

> [!QUESTION]
> I don't get why this metric of memory utilisation is important. Surely heap size is not what matters, but instead the amount of allocated memory? I am confused.
> In other words, why does it matter if we just [make our heap bigger](#^c52389)?

## Fragmentation
* The primary cause of poor heap utilisation is **fragmentation**.
* This occurs when otherwise unused memory is not available for allocation.
* There are two forms:

>* Internal Fragmentation
>* External Fragmentation

* **Internal fragmentation** occurs when an allocated block is larger than the payload. In other words, the amount of virtual memory allocated is greater than the amount requested by the program.
* This can happen for various reasons. For example, as we saw in Figure 9.34(b), the allocator might increase block size in order to satisfy alignment constraints.
* It's straightforward to quantify. It is simply the sum of the differences between the size of the allocated blocks, and their payloads:

$$\sum_k(\textrm{payload}_k-\textrm{allocated}_k)$$

* **External Fragmentation** occurs when there *is* enough total free memory to satisfy an allocate request, but no single free block is large enough to handle it.
* For instance, if the request in [Figure 9.34(e)](#^30899d) were for 8 words instead of 2 words, then the request couldn't be satisfied without requesting additional virtual memory from the kernel, even though there are 8 free words in the heap. The problem arises since these 8 words are spread over 2 free blocks.
* External fragmentation is harder to quantify. It depends not only on the pattern of previous requests and the allocator implementation, but also on the pattern of *future* requests.
* Allocators typically employ heuristics that attempt to maintain smaller numbers of larger free blocks, rather than large numbers of smaller free blocks.

## Implementation Issues
* The simplest imaginable allocator would organise the heap as a large array of bytes and a pointer `p` that initially points to the first byte of the array. ^c52389
* To allocate `size` bytes, `malloc` would save the current value of `p` on the [the stack](Educative%20Course.md#The%20Stack), increment `p` by `size`, and return the old value of `p` to the caller.
* This naive allocator is an extreme point in the design space. Throughput would be extremely good, since each `malloc` and `free` execute only a handful of instructions.
* However, since the allocator never reuses any blocks, memory utilisation would be extremely bad. A practical allocator that strikes a better balance between throughput and utilisation must consider the following:

> * **Free block organisation**: How do we keep track of free blocks?
> * **Placement**: How do we choose a free block to put a newly allocated block?
> * **Splitting**: After we place a newly allocated block in a free block, what do we do with the remainder of the block?
> * **Coalescing**: What do we do with a block that has just been freed?

The rest of this section looks at these issues in more detail. 
Since the basic techniques of placement, splitting, and coalescing apply to many different free block organisations, we'll introduce them in the context of a simple free block organisation: an implicit free list.

## Implicit Free Lists
* Any practical allocator needs some data structure that allows it to distinguish block boundaries and to distinguish between allocated and free blocks. 
* Most allocators embed this information in the blocks themselves. One simple approach is shown below:

![](_attachments/Screenshot%202023-05-14%20at%2010.57.05.png)

* A block consists of a one-word **header**, the payload, and possibly some **padding**.  ^099f6d
* The header encodes the block size, as well as whether the block is allocated or free. ^db2ce3
* If we impose a double-word alignment constraint; the block size is always a multiple of 8, so the 3 low-order bits of the block size are always 0. Thus, we need to store only the 29 high-order bits of the block size, freeing the remaining 3 bits to encode other information.[^fn1]
* In this case, we use the least significant of these bits to indicate whether the block is allocated or free. Hence, it is called the **allocated bit**.
* Given this block format, we can organise the heap as a sequence of contiguous allocated and free blocks, as shown in Figure 9.36: ^00057d

![](_attachments/Screenshot%202023-05-14%20at%2011.04.53.png)

* This organisation is called an **implicit free list** because the free blocks are linked implicitly by the size fields in the headers. The allocator can indirectly traverse the entire set of free blocks by traversing *all* the blocks in the heap.
* Notice that we need a specially marked end block - in this example, a terminating header with the allocated bit set, and a size of zero.
* The **advantage** of an implicit free list is simplicity. A **disadvantage** is the cost of any operation that requires a search of the free list, such as placing allocated blocks, will be linear in the **total** number of allocated and free blocks in the heap.
* It's important to realise that the system's alignment requirement and the allocator's choice of block format impose a **minimum block size** on the allocator. No allocated or free block may be smaller than this minimum.

## Placing Allocated Blocks
* When a program requests a block of $k$ bytes, the allocator searches the free list for a free block that is large enough to hold the requested block.
* The manner in which the allocator performs this search is determined by the **placement policy**.
* Some common policies are:
> * First fit
> * Next fit
> * Best fit ^7417a3

* **First fit** searches the free list from the beginning, and chooses the first free block that fits.
* **Next fit** is similar, but instead of starting each search at the beginning of the list, it starts from where the previous search left off.
* **Best fit** examines every block and chooses the free block with the smallest size that fits.
* An advantage of first fit is that it tends to retain large free blocks at the end of the list. A disadvantage is that it tends to leave "splinters" of small free blocks toward the beginning of the list, which will increase the search time for larger blocks.
* Next fit is motivated by the idea that if we found a fit in some free block the last time, there is a good chance that we'll find a fit the next time in the remainder of the block.
* Next fit can run significantly faster than first fit, especially if the front of the list has many splinters. However, some studies suggest it has worse memory utilisation than first fit.
* Best fit usually has better memory utilisation. But using it with simple free list organisations such as the implicit free list requires an exhaustive search of the heap. 
* There exist more sophisticated segregated free list organisations that approximate a best-fit policy without an exhaustive search of the heap.

## Splitting Free Blocks
* Once the allocator has located a free block that fits, it must make another policy decision about how much of the free block to allocate.
* One option is to use the **entire free block**. This is simple and fast, but introduces internal fragmentation. This might not be a problem if the placement policy produces good fits.
* However, if the fit is not good, then the allocator will usually opt to **split** the free block into two parts. The first becomes the allocated block; the second becomes a new free block.
* Figure 9.37 shows how an allocator might split the eight-word free block in [Figure 9.36](#^00057d) to satisfy a request for three words of heap memory: ^0d6c04

![](_attachments/Screenshot%202023-05-14%20at%2011.23.11.png)

* Notice how the allocated block is 4 words in size; the extra word is for the block header.

## Getting Additional Heap Memory
* What happens if the allocator is unable to find a fit for the requested block?
* One option is to create some larger free blocks by merging ([coalescing](#Coalescing%20Free%20Blocks)) free blocks that are physically adjacent in memory.
* However, if this doesn't yield a sufficiently large block, or if the free blocks are already maximally coalesced, then the allocator asks the kernel for additional heap memory.
* It does so with the `sbrk` function.
* The allocator transforms the additional memory into one larger free block, inserts the block into the free list, and places the requested block in this new free block.

## Coalescing Free Blocks
* When the allocator frees an allocated block, there might be other free blocks that are adjacent to the newly freed block.
* This can cause a phenomenon called **false fragmentation**, where there is a lot of available free memory chopped up into small, unusable free blocks.
* As an example, Figure 9.38 shows the result of freeing the block that was allocated in [Figure 9.37](#^0d6c04):

![](_attachments/Screenshot%202023-05-14%20at%2011.32.34.png)

* The result is two adjacent free blocks with payloads of three words each.
* As a result, a subsequent request for a payload of 4 words would fail, even the the aggregate size of the two free blocks is large enough to satisfy the request.
* To combat this, any practical allocator merges adjacent free blocks in a process called **coalescing**.
* This raises an important policy decision about when to perform coalescing. The allocator can opt for **immediate coalescing** by merging any adjacent blocks each time a block is freed. Or it can opt for **deferred coalescing** by waiting until some later time.
* For instance, the allocator might defer coalescing until some allocation request fails, and then scan the entire heap, coalescing all free blocks.
* Immediate coalescing is straightforward and can be performed in constant time, but with some request patterns it can introduce a form of [thrashing](Cache%20Memories#^2073ef) where a block is repeatedly coalesced and split.


> [!INFO]
> I skipped the remaining content of this section. The two main parts are:
> * Coalescing with Boundary Tags
> * Implementing a Simple Allocator (in C)












[^fn1]: We use 32 bits to encode the block size, since this is our word size.
[^fn2]: This description is quite common, but a bit confusing. Memory isn't *actually* allocated at compile time. See [here](https://stackoverflow.com/questions/8385322/difference-between-static-memory-allocation-and-dynamic-memory-allocation) for more information.






