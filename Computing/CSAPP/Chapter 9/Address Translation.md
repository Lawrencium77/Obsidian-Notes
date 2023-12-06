This sections covers the basics of address translation. Keep in mind that we are omitting many details that are important to hardware designers but beyond our scope.

```toc
```

### Introduction
* Formally, address translation is a mapping a virtual address space (VAS) and physical address space (PAS).
* Figure 9.12 shows how the MMU uses a page table to do this:

![](_attachments/Screenshot%202023-04-10%20at%2015.23.53.png)
* A control register in the CPU - the **page table base register** (PTBR) - points to the page table.
* The above diagram looks a bit complicated, but really isn't.
* Each $n$-bit VA can be interpreted as having two components:
>* **Virtual page number** (VPN) - this tells you which page you're in.
>* **Virtual page offset** (VPO) - this tells you where in the page you are. 
* VPN 0 selects PTE 0, VPN 1 select PTE 1, etc.
* The corresponding PA is the concatenation of the PPN from the PTE, and the VPO from the virtual address.
* The **physical page offset** (PPO) is identical to the VPO.
* Figure 9.13(a) shows the steps that the CPU hardware performs when there is a **page hit**:

![](_attachments/Screenshot%202023-04-11%20at%2018.52.17.png)

> 1. The processor generates a VA and sends it to the MMU
> 2. The MMU generates the PTE address and requests it from cache / MM (i.e. it asks the page table for the PTE).
> 3. The cache / MM returns the PTE to the MMU
> 4. The MMU constructs the physical address and sends it to cache / MM.
> 5. The cache / MM returns the requested data word to the processor.

* This is handled entirely by the hardware.
* Handling a **page fault** requires cooperation between the hardware and the OS kernel: ^d0e7a7

![](_attachments/Screenshot%202023-04-11%20at%2018.55.31.png)

> 1-3. The same as above.
> 4. The valid bit in the PTE is zero. The MMU triggers an exception, which transfers control in the CPU to a page fault exception handler in the OS kernel.
> 5. The fault handler identifies a victim page in MM, and if that has been modified, pages it out to disk. 
> 6. The fault handler pages in the new page and updates the PTE in memory.
> 7. The fault handler returns control to the original process. The CPU resends the offending VA to the MMU. We now have a cache hit.

### Integrating Caches and VM
* In a system that uses both VM and SRAM caches, we must decide whether to use VAs of PAs to access the SRAM cache.
* Most systems opt for physical addressing.
* This makes it easy for multiple processes to have blocks in the cache at the same time and to share blocks from the same VPs.
* The main idea is that in a physically addressed cache, the address translation occurs before cache lookup:

![](_attachments/Screenshot%202023-04-11%20at%2019.11.54.png)

* Notice that PTEs can be cached, just like any other data words.

> [!QUESTION]
> In Figure 9.14, the cache is described as being *physically* addressed, while the main memory is *virtually* addressed. Yet address translation happens prior to lookup in both cases, and I don't actually see any difference whatsoever. Where's the hole in my understanding?

### Speeding Up Address Translation with a TLB
* Every time a CPU generates a VA, the MMU must refer to a PTE to do address translation.
* This could a fetch from main memory. The PTE might be cached in L1, which would speed things up.
* However, many systems try to eliminate even this cost by including a small cache of **PTEs** in the MMU called a **translation lookaside buffer (TLB)**.
* A TLB is a small cache where each line holds a block containing a **single PTE**.
* The index and tag fields that are used for set selection and line matching are extracted from the VPN in the VA[^fn1]:

![](_attachments/Screenshot%202023-04-11%20at%2019.16.53.png)

* Figure 9.16(a) shows the steps when there is a TLB hit. The key point is that all the address translation steps are performed inside the MMU (unlike Figure 9.13(a)):

![](_attachments/Screenshot%202023-04-11%20at%2019.18.54.png)

> 1. CPU generates a VA.
> 2. MMU fetches the PTE from the TLB.
> 3. (Same as 2)
> 4. MMU translates the VA to a PA and sends it to cache / MM.
> 5. Cache / MM returns the requested word to the CPU.

When there is a TLB miss, the MMU must fetch the PTE from L1 cache. The newly fetched PTE is stored in the TLB, possibly evicting an existing entry:

![](_attachments/Screenshot%202023-04-11%20at%2019.20.54.png)


### Multi-Level Page Tables
* So far, we've assumed the system uses a single page table to do address translation.
* But if we had a 32-bit address space, 4 KB pages, and a 4-byte PTE (since our address space is 32 bits). We'd need a page table of size:

$$\textrm{PTE size} \times \textrm{Number of Pages}=4 \space\textrm{bytes}\times2^{32} \space \textrm{bits}/(4 \space \textrm{KB})=4\space \textrm{MB}$$

* This would just be resident in MM the whole time.
* The common solution to compact the page table is to use a **hierarchy** of page tables instead.
* Let's consider a concrete example. Consider a 32-bit VAS, 4 KB pages, with PTEs of 4 bytes. 
* Suppose the first 2 K pages of memory are allocated for code and data, the next 6 K pages are unallocated, the next 1023 are also unallocated, and the next page is allocated for the user stack:

![](_attachments/Screenshot%202023-04-11%20at%2019.29.12.png)

* Each PTE in the level 1 table is responsible for mapping a 4 MB chunk of the VAS, where each chunk consists of 1024 pages.
* Since the address space is 4 GB, 1024 PTEs cover the entire address space.
* If at least one page in chunk $i$ is allocated, then level 1 PTE $i$ points to the base of a level 2 page table.
* Each PTE in a level 2 page table maps a 4 KB page of VM, just as before (when looking at single-level page tables).
* This scheme reduces memory requirements in two ways:

> 1. If a PTE in the level 1 table is null, the corresponding level 2 page table **doesn't have to exist**. We don't even have to allocate it.
> 2. Only the level 1 table needs to be in MM at all times. The level 2 tables can be paged in and out of the VM system as they're needed.

> [!INFO]
> We can interpret this 2-level page table as splitting the virtual address space into different "regions", each containing $2^{10}$ pages. 
> Each region corresponds to a PTE in the level 1 page table.
> There are $2^{10}$ such regions:
![|500](_attachments/Screenshot%202023-04-14%20at%2016.41.38.png)

* We can generalise this to use a $k$-level page table hierarchy:

![|500](_attachments/Screenshot%202023-04-14%20at%2016.42.47.png)

* Each VA is partitioned into $k$ VPNs and a VPO. 
* Each VPN is an index into a page table at level $i$.
* Each PTE in a level $j$ page table, $1 \le j \le k-1$ points to the base of same page table at level $j+1$.
* Each PTE in a level $k$ table contains either the PPN of some physical page, or the address of a disk block.
* To construct the physical address, the MMU must access $k$ PTEs before it can determine the PPN. 
* This may seem impractical at first glance. But the TLB comes to the rescue by caching the PTEs from page tables at different levels. In practice, using a hierarchical page table is not significantly slower than a single-level page table.


### Putting It Together: End-to-End Address Translation
I didn't make notes on this section. But it is a good read to help review all the address translation content.






[^fn1]: This is obviously the case since the VPN is used as to index into a page table.