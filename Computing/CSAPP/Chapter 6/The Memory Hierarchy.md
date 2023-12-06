* In sections 6.1 and 6.2 we saw that:
	* Faster storage technologies are smaller, and more expensive.
	* Well-written programs exhibit good locality.
* These fundamental properties of hardware and software suggest an approach for organising memory systems, called the **memory hierarchy**:

![](_attachments/Screenshot%202023-03-24%20at%2021.16.24.png)

* At the highest level, the CPU can access its registers in a single clock cycle.
* An example of the bottom level is the **Network File System (NFS)**. This is a distributed file system that allows a program to access file stored on remote network-connected servers.

```toc
```

### Caching in the Memory Hierarchy
* In general, a **cache** is a small, fast storage device that acts as a staging area for the data stored in a larger, slower device.
* The fundamental idea of the **memory hierarchy** is that level $k$ serves as cache for level $k+1$. Here's an illustration:

![](_attachments/Screenshot%202023-03-24%20at%2021.37.17.png)

* Level $k+1$ is partitioned into contiguous chunks of data called **blocks**. 
* Level $k$ is partitioned into blocks that are the same size as in $k+1$. There are fewer of them.
* Data are always copied between levels in block-size transfer units.
* Note: block size is fixed between any two adjacent levels, but other pairs of levels can have different block sizes. 
* E.g., L0 and L1 typically use word-size blocks. But L1 and L2 use blocks of tens of bytes.

#### Cache Hits
* When a program needs a data object $d$ from level $k+1$, it first looks in the blocks stored at $k$.
* If $d$ is cached at level $k$, this is a **cache hit**.

#### Cache Misses
* If $d$ is not cached at level $k$, it's called a **cache miss**.
* In this case, the cache at level $k$ fetches the block from $k+1$, possibly overwriting an existing block if the level $k$ cache is full.
* This is known as **replacing** or **evicting** the block.
* The block that's evicted is the **victim block**.
* The decision on which block to replace is called the **cache replacement policy**.

#### Kinds of Cache Misses
* It's useful to distinguish between kinds of cache misses.
* [Cold misses](https://en.wikipedia.org/wiki/Cache_performance_measurement_and_metric#Compulsory_misses): ^83b8d3
	* Each memory block, when first referenced, results in a **compulsory miss**. They're also called **cold misses**.
	* This implies the number of cold misses is the number of distinct memory blocks ever referenced.
	* They cannot be avoided unless the block is the data is [prefetched](https://en.wikipedia.org/wiki/Cache_prefetching).
	* They're transient events that don't occur once the cache has been **warmed up**.
* [Placement Policy](https://en.wikipedia.org/wiki/Cache_placement_policies#:~:text=In%20other%20words%2C%20the%20cache,associative%2C%20and%20set%2Dassociative.):
	* When there's a miss, the cache at level $k$ must implement a **placement policy**[^fn1] that determines where to place the block retrieved from $k+1$.
	* The most flexible policy is to allow any block from $k+1$ to be stored in any block at level $k$.
	* High in the memory hierarchy, where speed is at a premium, this is too expensive because randomly placed blocks are expensive to locate.
	* Thus, hardware caches typically implement a simpler placement policy that restricts a particular block at level $k+1$ to a subset of the blocks at level $k$.
	* E.g. a block $i$ at level $k+1$ might be mapped to block $(i\mod 4)$ at level $k$.
	* Blocks 0,3,8,12 would map to block 0, and so on. The example in Figure 6.22 does this.
* [Conflict misses](https://en.wikipedia.org/wiki/Cache_performance_measurement_and_metric#Conflict_misses): ^0a4490
	* **Restrictive placement policies** of this kind lead to a type of miss called a **conflict miss**. The cache is big enough to hold the referenced data, but because they map to the same cache block, the cache keeps missing.
	* E.g. in Figure 6.22, in a program that requests block 0, then 8, then 0, and so on, each of the references to these two blocks would miss. Despite the cache being able to hold 4 blocks.
* Programs often run as a sequence of phases (e.g. loops) where each phase accesses some reasonably constant set of cache blocks. E.g. a nested loop might access the elements of the same array over and over again.
* This set of blocks is called the **working set** of the phase. ^84399c
* When the size of the working set exceeds the cache size, we observe [capacity misses](https://en.wikipedia.org/wiki/Cache_performance_measurement_and_metric#Capacity_misses). The cache is too small to handle the working set.

#### Cache Management
* At each level of the memory hierarchy, some logic must **manage** the cache.
* Something has to partition the cache into blocks, transfer blocks between levels, and deal with cache misses. 
* This logic can be hardware, software, or a combination.
* For instance, the compiler manages the register file. It decides when to issue loads when there are misses, and which register to store the data in.
* L1, L2, and L3 caches are managed entirely by built-in hardware logic.
* In a system with virtual memory, the DRAM main memory serves as cache for disk storage. It's managed by a combination of OS software and address translation hardware on the CPU.

### How Caches Exploit Temporal and Spatial Locality
This info is in the summary section. I thought it's nice to clarify:

* **Temporal locality** means the same data objects are likely to be reused multiple times. Once a data object has been copied into the cache after the first miss, we can expect a number of subsequent hits on that object.
* Blocks usually contain multiple data objects. Because of **spatial locality**, we can expect the cost of copying a block after a miss will be amortised by subsequent references to objects within that block.



[^fn1]: Not to be confused with *cache replacement policies*.