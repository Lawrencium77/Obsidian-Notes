For reference, the typical bus structure for cache memories is shown here:

![](_attachments/Screenshot%202023-03-28%20at%2021.54.40.png)

```toc
```

## Generic Cache Memory Organisation
* Consider a system with word size $m$. Each address has $m$ bits. 
* Figure 6.25(a) shows how a cache for such a machine is organised:

![](_attachments/Screenshot%202023-03-28%20at%2022.05.37.png)

The contents of this diagram can be explained as follows:
* A cache is as an array of $S=2^s$ **cache sets**.
* Each set consists of $E$ **cache lines**.
* Each line consists of a **valid bit**, a **tag bit**, and a **block** of data.
* A valid bit indicates whether or not the line contains meaningful information.
* There are $t=m-(b+s)$ tag bits. These uniquely identify the block stored on the cache line.
* Each block has $B=2^b$ bytes.

In general, a cache's organisation can be characterised by the tuple $(S,E,B,m)$.

* The size of a cache, $C$, is stated as the summed size of all the blocks. The tag and valid bits aren't included. Hence, $C=S\times E\times B$. 
* The parameters $S$ & $B$ partition the $m$ address bits into the three fields shown in Figure 6.25(b).

On a high level, this operates as follows:

* When the CPU is instructed to read a word from address $A$ of **main memory**, it sends $A$ to the cache (to see if the word is there, before going to main memory).
* If the cache is holding a copy of the word at $A$, it sends the word immediately back to the CPU.
* How does the cache know whether it contains a copy of the word at $A$?
* The cache is organized so it can find the requested word by inspecting the bits of the address. It's similar to a hash table with a very simple hash function:
	* The $s$ **index bits** in $A$ form an index into the array of $S$ sets. They tell us which set the word must be stored in.
	* The $t$ **tag bits** tell us which line in the set contains the word. A line in the set contains the word if and only if the valid bit is set and the tag bits in the line match the tag bits in $A$.
	* Once we've located the line, the $b$ **block offset bits** give us the offset of the word in the $B$-byte data block.

## Direct-Mapped Caches
* Caches are split into different classes based on $E$ - the number of lines per set.
* A cache with one line per set ($E=1$) is called a **direct-mapped cache**:

![](_attachments/Screenshot%202023-03-28%20at%2022.19.31.png)

* They're the simplest to implement and understand. So we'll use them to illustrate some concepts about how caches work.
* Suppose we've a system with a CPU, a register file, an L1 cache, and a main memory. When the CPU tries to read a memory word $w$, it does the following:
	* Requests $w$ from L1 cache.
	* If L1 has $w$, it returns to the CPU.
	* Else, the CPU waits for L1 cache to request a copy of $w$ from memory. L1 cache stores $w$ in one of its lines, extracts $w$ from the stored block, and returns it to the CPU.
* This can be split into three different steps:
	* (1) **Set Selection**
	* (2) **Line Matching**
	* (3) **Word Extraction**

#### Set Selection in Direct-Mapped Caches
* In this step, the cache extracts set index $s$ from the middle of the address for $w$.
* These bits are interpreted as an unsigned int that corresponds to a set number.
* Figure 6.28 shows an example:

![](_attachments/Screenshot%202023-03-28%20at%2022.25.07.png)

#### Line Matching in Direct-Mapped Caches
* Now that we've selected a set $i$, the next step is to determine if a copy of $w$ is stored in one of its cache lines.
* In direct-mapped cache, this is easy since there is one line per set.
* A copy of $w$ is contained if and only if the valid bit is set, and the tag in the cache line matches the tag in the address of $w$.
* Figure 6.29 shows how line matching works in a direct-mapped cache:

![](_attachments/Screenshot%202023-03-28%20at%2022.26.51.png)

#### Word Selection in Direct-Mapped Caches
* Once we have a hit, we know that $w$ is somewhere in the block.
* The last step determines where the desired word starts the block. 
* Figure 6.29 shows this too. It assumes a word is 4 bytes long.

#### Line Replacement on Misses in Direct-Mapped Caches
* If the cache misses, it needs to retrieve the requested block from the next level of memory hierarchy and store the new block in one of its cache lines, indicated by the set index bits.
* If the set is full of valid cache lines, then one of them must be evicted.
* For a direct-mapped cache, where each set contains one line, the replacement policy is trivial: the current line is replaced by the newly fetched line.

#### A Direct-Mapped Cache in Action
* Hopefully, a concrete example helps clarify the process.
* Suppose we have a direct-mapped cache described by:

$$(S,E,B,m)=(4,1,2,4)$$
* In other words, we've 4 sets, 1 line per set, 2 bytes per block, and 4-bit addresses.
* We assume each word is a single byte.
* It's instructive to enumerate the address space and partition its bits:

![](_attachments/Screenshot%202023-03-29%20at%2021.29.18.png)

* There are some useful things to notice:
	* The combination of the tag & index bits uniquely identifies each block in memory.
	* Multiple blocks map to the same cache set.
	* Blocks that map to the same set are uniquely identified by the tag.
* Let's simulate the cache in action as the CPU performs a sequence of reads. Remember that we assume the CPU reads 1-byte words:
* Initially, the cache is empty (each valid bit is 0):
	
![|500](_attachments/Screenshot%202023-03-29%20at%2021.31.35.png)

* Each row in the table represents a cache line.
* Let's see what happens when the CPU performs a sequence of reads:

1. **Read word at address 0**. Since the valid bit is 0, this is a cache miss. The cache fetches block 0 from memory, and stores it in block 0. The cache returns `m[0]` (the contents of memory location 0):

![|500](_attachments/Screenshot%202023-03-29%20at%2021.35.40.png)
2. **Read word at address 1**. This is a cache hit. The cache returns `m[1]`.
3. **Read word at address 13 (0b1101)**. Since the cache line in set 2 is not valid, this is a miss. The cache loads block 6 into set 2 and returns `m[13]` from `block[1]` of the new cache line.

![|500](_attachments/Screenshot%202023-03-29%20at%2021.40.04.png)
4. **Read word at address 8 (0b1000)**. This is a cache miss. The cache line in set 0 is valid, but the tags don't match. The cache loads block 4 into set 0 (replacing the previous line) and returns `m[8]`:

![|500](_attachments/Screenshot%202023-03-29%20at%2021.39.51.png)

5. **Read word at address 0**. This is another miss, since we just replaced block 0. This is an example of a [conflict miss](The%20Memory%20Hierarchy#^0a4490).

#### Conflict Misses in Direct-Mapped Caches
* Conflict misses are common in real programs.
* In direct-mapped caches, they typically occur when programs access arrays whose sizes are a power of 2. For instance:

```C
float dotprod(float x[8], float y[8])
{
	float sum = 0.0;
	int i;

	for (i = 0, i < 8; i++)
		sum += x[i] * y[i];
	return sum;
}
```

* This function has good spatial locality w.r.t `x` and `y`, so we might expect a good number of cache hits. But this isn't always true.
* Suppose that floats a 4 bytes, that `x` is loaded into 32 bytes of contiguous memory starting at address 0, and `y` starts immediately after at address 31.
* For simplicity, assume a block can hold 16 bytes (big enough for four floats), and that the cache consists of two sets - a total cache size of 32 bytes.
* We assume `sum` is stored in a CPU register and so doesn't need a memory reference.
* Given these assumptions, each `x[i]` and `y[i]` will map to the same cache set:

![](_attachments/Screenshot%202023-03-29%20at%2021.48.16.png)

* First loading `x[0]` results in a [cold miss](The%20Memory%20Hierarchy#^83b8d3). The same goes for `x[4]`. 
* Other than these reads, every single read results in a conflict miss.
* The term [thrashing](https://www.quora.com/What-is-cache-thrashing) can be applied here: it describes any situation where a cache is repeatedly loading and evicting the same sets of cache blocks. ^2073ef
* The bottom line is that even though the program has good spatial locality, and we have room in the cache to hold the blocks for both `x[i]` and `y[i]`, each reference results in a conflict miss.
* Luckily, thrashing is easy to fix. One easy solution is to put `B` bytes of padding at the end of each array.

> [!INFO]
> #### Why index with the middle bits?
> You may wonder why caches use the middle bits for the set index. 
> There is good reason. If the high-order bits are used as an index, contiguous memory blocks will map to the same cache set.
> Figure 6.31 shows this. If a program has good spatial locality, high-order bit indexing means that the cache only holds a block-size chunk of the array at any one time. We'd get several conflict misses.
> In middle-bit indexing, the cache can hold an entire $C$-size chunk of the array, where $C$ is the cache size.

![](_attachments/Screenshot%202023-03-29%20at%2021.58.54.png)

## Set Associative Caches
* A **set associative cache** has $1<E<C/B$ cache lines per set.
* A cache with $E$ is often called an $E$-way set associative cache.
* Figure 6.32 shows an example with $E=2$:

![](_attachments/Screenshot%202023-03-29%20at%2022.03.06.png)

#### Set Selection
* Is identical to the process used in a direct-mapped cache.

#### Line Matching and Word Selection 
* A reminder: line-matching is the process of checking if a word is already stored in the cache.
* This is more involved that in direct-mapped caches, since it must check the tags and valid bits of multiple lines.
* We can think of each set in a set associative cache as an [associative memory](../../Conventional%20and%20Associative%20Memory.md). They keys are the concatenations of the tag & valid bits; the values are the contents of each block.
* Figure 6.34 shows the basic idea of line matching in a set associative cache:

![](_attachments/Screenshot%202023-04-02%20at%2011.42.34.png)

* **Important idea:** any line in the set can contain any of the memory blocks that map to that set.
* So the cache must search each line in the set.

#### Line Replacement 

^2fa447

* Upon a cache miss, the cache must fetch the block that contains the word requested by the CPU.
* But which line should it replace?
* If there's an empty line, then it should replace this. Otherwise, it's not obvious.
* It's very hard for programmers to exploit details on cache replacement policies, so we won't go into too much detail here.
* Such policies require time and hardware to compute. But as we move down the memory hierarchy, the cost of a miss becomes more expensive. So it becomes more worthwhile to minimise misses with good replacement policies.

## Fully Associative Caches
* A **fully associative cache** consists of a single set that contains all of the cache lines.
* Figure 6.35 shows the basic organisation:

![](_attachments/Screenshot%202023-04-02%20at%2011.47.29.png)

#### Set Selection 
* This is trivial, since there's only one set.

#### Line Matching and Word Selection 
* This works the same as with a set associative cache, as shown in Figure 6.37:

![](_attachments/Screenshot%202023-04-02%20at%2011.48.50.png)
* The main difference is one of scale. Because the cache circuitry must search for many tags in parallel, it's difficult and expensive to build an associative cache that is large and fast.
* So fully associative caches are only appropriate for small caches.


## Issues with Writes
* The operation of a cache with respect to reads is straightforward.
* Writes are more complicated.
* Suppose the processor wishes to write a word $w$ to a block of data. If the block is present in a cache, we have a **write hit**. In other words, the cache contains the desired word to be modified.
* After the cache updates its copy of $w$, what does it do about updating the copy of $w$ in the next lower level of hierarchy?
* The simplest approach is called **write-through**: ^31fa6d
	* This immediately writes $w$'s cache block to the next lowest level. 
	* While simple, this causes bus traffic with every write.
* Another is **write-back**: ^56d98c
	* This defers the update as long as possible by writing the updated block to the next lowest level only when it's evicted from the cache.
	* Due to locality, this can reduce bus traffic.
	* But it has the disadvantage of extra complexity.
	* The cache must maintain an additional **dirty bit** for each cache line. This indicates whether or not the cache block has been modified.
* Another issue is how to deal with **write misses**. 
* One approach is called **write-allocate**:
	* This loads the corresponding block from the next lowest level into the cache, and updates the cache block.
	* This tries to exploit the spatial locality of writes.
	* But means every miss results in a block transfer.
* The alternative is **no-write-allocate**:
	* This bypasses the cache and writes directly to the next-lowest level.
* Write through caches are typically no-write-allocate.
* Write-back caches are typically write-allocate.
* Optimising caches for writes is subtle and difficult. We are only scratching the surface here.
* For the programmer trying to write reasonably cache-friendly code, the authors recommend adopting a mental model that assumes **write-back, write-allocate** caches.

> [!INFO]
> The [blog](https://github.com/Lawrencium77/Blog-Timing-CUDA-Kernels) I wrote with David mentions approaches to writes in caches. The key point is that L2 cache on NVIDIA GPUs use a write-back policy. 

## Anatomy of a Real Cache Hierarchy
* So far.  we assumed caches only hold program data.
* But they can also hold instructions.
* A cache that holds **instructions only** is called an **i-cache**.
* A cache that holds **program data only** is called a **d-cache**.
* A cache that holds both is called a **unified cache**.
* Modern processors include separate i-caches and d-caches. The advantages of this are:
	* With two separate caches, the processor can read an instruction word and a data word at the same time.
	* I-caches are typically read-only, so are simpler.
	* They can be optimised for different access patterns.
* Figure 6.38 shows that cache hierarchy of the Intel Core i7 processor. Each chip has 4 cores:

![](_attachments/Screenshot%202023-04-02%20at%2011.57.05.png)

## Performance Impact of Cache Parameters
* Cache performance is evaluated with a number of metrics:
	* **Miss Rate** - the number of references during program execution that miss. $$\textrm{Miss Rate}=\frac{\textrm{Num Misses}}{\textrm{Num References}}$$ ^240296
	* **Hit Rate** - the inverse of miss rate.
	* **Hit time** - the time to deliver a word in the cache to CPU. This includes time for set selection, line identification, and word selection.
	* **Miss penalty** - Any additional time required because of a miss.

We can identify some qualitative cost-performance trade-offs of cache memories.

#### Impact of Cache Size
* A larger cache increases hit rate.
* But it's harder to make large memories run faster. So it increases the hit time too.

#### Impact of Block Size
* Large blocks can increase hit rate, by exploiting spatial locality.
* But for a given cache size, larger blocks imply a smaller number of cache lines $\implies$ can hurt hit rate for programs with more temporal than spatial locality.
* Larger blocks also increase the miss penalty, since it takes longer to transfer them.

#### Impact of Associativity
* This concerns the choice of $E$.
* Higher associativity decreases the vulnerability of thrashing.
* But it's more expensive to implement, and hard to make fast.
* It can increase hit time, due to increased complexity.
* It can increase miss time, due to complexity of choosing a victim line.

#### Impact of Write Strategy
* Write-through caches are simpler to implement. They can use a **write buffer** that works independently of the cache to update memory.
* Furthermore, read misses are less expensive since they don't trigger a memory write.
* On the other hand, write-back caches result in fewer transfers, allowing more bandwidth to memory for I/O devices that perform DMA.
* Moreover, reducing the number of transfers becomes increasingly important as we move down the hierarchy since transfer times increase.
* In general, caches further down the hierarchy are more likely to use write-back than write-through.

## Cache Line vs Set vs Block
CSAPP makes the distinction between caches lines, set, and blocks:

>A **block** is a fixed-size packet of data that moves between adjacent caches.
>A **line** is a container that stores a block, as well as other information such as the valid and tag bits.
>A **set** is a collection of one or more lines. 

In direct-mapped caches, sets and lines are equivalent.
Since a line always stores a single block, "line" and "set" are often used interchangeably. This usage is very common and shouldn't cause any confusion provided you understand the difference between blocks and lines.