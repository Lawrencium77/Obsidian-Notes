```toc
```

### Introduction
* Conceptually, a VM is organised as an array of $N$ contiguous byte-size cells stored **on disk**.
* The contents of the array on disk are **cached in main memory**.
* As with all caches, the data on disk (the lower level) is partitioned into [blocks](The%20Memory%20Hierarchy#Caching%20in%20the%20Memory%20Hierarchy) that serve as transfer units between the disk and main memory (the upper level).
* In a VM system, each block is called a **virtual page** (VP). ^27a1b0
* Each VP contains $P=2^p$ bytes.
* Similarly, physical memory is partitioned into **physical pages** (PP), also $P$ bytes in size.
* At any point in time, the set of virtual pages is partitioned into three disjoint subsets:

> * **Unallocated** - Pages that have not yet been allocated by the VM system. Unallocated blocks do not have any data associated with them, and thus **don't occupy any space on disk**.
> * **Cached** - allocated pages that are currently cached in main memory.
> * **Uncached** - allocated pages that are not cached.

* The following diagram gives an example:

![](_attachments/Screenshot%202023-04-07%20at%2014.43.37.png)

* We see a virtual memory with 8 pages. VP 0 and 3 haven't been allocated yet, and thus do not exist on disk.
* VPs 1, 4, and 6 are cached in physical memory.
* Pages 2,5 and 7 are uncached.

> [!INFO]
> Whilst addresses that are contiguous in virtual memory need not be contiguous in physical memory, memory *within* a single VM page must be contiguous in physical memory. This makes a lot of sense when we consider [Address Translation](Address%20Translation.md). See [source](https://stackoverflow.com/questions/22020868/virtually-contiguous-vs-physically-contiguous-memory).

### DRAM Cache Organisation
* To be clear: we use **SRAM Cache** to denote L1, L2, and L3 cache memories.
* We use **DRAM Cache** to denote the VM system's cache that caches virtual pages in main memory.
* DRAM **cache misses** are very **expensive**, since reading from disk is about 100,000x slower than reading from DRAM.
* DRAM cache organisation is driven entirely by the enormous cost of misses.
* It motivates large virtual pages - typically 4 KB to 2 MB.
* The large miss penalty means that DRAM caches are [fully associative](Cache%20Memories#Fully%20Associative%20Caches); any virtual page can be placed in any physical page.[^fn1]
* The **replacement policy** policy on misses is also very important, because the penalty associated with replacing the wrong virtual page is so high. OSs use more sophisticated [replacement policies](Cache%20Memories#^2fa447) for DRAM caches than the hardware does for SRAM caches.
* Because of the large access time of disk, DRAM caches are [write-back](Cache%20Memories#^56d98c) instead of [write-through](Cache%20Memories#^31fa6d).

### Page Tables

^d48eb6

* As with any cache, the VM system needs a way to determine if a VP is cached in DRAM.
* If so, the system must determine which PP it is cached in.
* If there's a miss, the system must determine where the VP is stored on disk, select a victim page in physical memory, and copy the VP from disk to DRAM.
* These capabilities are provided by a combination of:

> * OS Software
> * Address translation hardware in the MMU
> * A data structure stored in physical memory called the **page table**, that maps VPs to PPs.

* The address translation hardware reads the page table each time it converts a virtual address to a physical address.
* The OS is responsible for maintaining the page table, and transferring pages between disk and DRAM.
* The following diagram shows the basic organisation of a page table:

![](_attachments/Screenshot%202023-04-07%20at%2015.02.48.png)

* A page table is an array of **page table entries (PTEs)**.
* Each page in the virtual address space has a PTE at a fixed offset in the page table.
* For our purposes, we assume that each PTE consists of a **valid bit** and an $n$-bit address field. 
* The valid bit indicates whether the VP is currently cached in DRAM.
* If the valid bit is set, the address field indicates the start of the corresponding physical page in DRAM (where the VP is cached).
* If it is not set, then a null address indicates the VP has not yet been allocated. Otherwise, the address points to the start of VP on disk.
* The above diagram shows a page table for a system with 8 VPs and 4 PPs. 
* 4 VPs are currently cached in DRAM.
* 2 have not yet been allocated.
* The rest have been allocated but are not cached.
* Note: because the DRAM cache is fully associative, any PP can contain any VP.

### Page Hits
* Consider what happens when the CPU reads a word of VM contained in VP 2, which is cached in DRAM:

![](_attachments/Screenshot%202023-04-07%20at%2015.14.22.png)

* Using a technique described in the section on [address translation](Address%20Translation.md), the address translation hardware uses the virtual address as an index to locate PTE 2 (which points to PP 1) and read it from memory.
* Since the valid bit is set, it knows that VP 2 is cached in MM. So it uses the physical memory address in the PTE to construct the physical address of the word.

### Page Faults
* In VM parlance, a [cache miss](The%20Memory%20Hierarchy#Cache%20Misses) is called a **page fault**.
* Figure 9.6 shows the state of a page table before the fault:

![](_attachments/Screenshot%202023-04-07%20at%2015.16.07.png)

* The CPU has referenced a word in VP 3, which is not cached in DRAM.
* The address translation hardware reads PTE 3 from memory, infers from the valid bit that VP 3 is not cached, and triggers a page fault **exception**.
* This invokes a page fault **exception handler** in the kernel, which selects a victim page - in this case, VP 4 stored in PP 3. The kernel also modifies the page table entry for VP 4 to reflect the fact that VP 4 is no longer cached in MM.
* Next, the kernel copies VP 3 from disk to PP 3 in MM, updates PTE 3, and then returns.
* When the handler returns, it restarts the faulting instruction, which resends the faulting virtual address to the address translation hardware. But now, VP 3 is cached in MM, and the page hit is handled normally by the address translation hardware.
* Figure 9.7 shows the state of our example page table after the page fault:

![](_attachments/Screenshot%202023-04-07%20at%2015.19.05.png)


### Allocating Pages
* Figure 9.8 shows how our example page table changers when the OS allocates a new page of VM. Notice that the number of VPs on disk grows by one:

![](_attachments/Screenshot%202023-04-07%20at%2015.28.34.png)

### Locality to the Rescue Again
* In practice, VM works well, mainly because of [locality](../Chapter%206/Locality.md).
* Although the total number of distinct pages that programs reference might exceed the total size of MM, locality suggests that at any point in time, they will tend to work on a smaller set of **active pages** (the [working set](The%20Memory%20Hierarchy#^84399c)).
* After an initial overhead where the working set is paged into memory, subsequent references result in hits with no additional disk traffic.

> [!INFO]
> VM was invented in the 1960s, before the widening CPU-memory gap spawned SRAM caches. As a result, VM systems use a different terminology from SRAM caches, even though many of the ideas are similar:
> * In VM, **blocks** are called **pages**.
> * In VM, transferring a page is called **swapping** or **paging**.
> * Pages are **swapped in** from disk to DRAM, and **swapped out** from DRAM to disk.
> * The strategy of waiting until the last moment to swap in a page, when a miss occurs, is called [demand paging](4%20-%20Processes#^c69d45). All modern systems use demand paging.


[^fn1]: I think this prevents the likelihood of conflict misses.