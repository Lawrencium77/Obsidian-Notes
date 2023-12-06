* In the [previous section](VM%20as%20a%20Tool%20for%20Caching.md), we assumed a single page table that maps a single virtual address space to the physical address space.
* In fact, OSs provide a **separate page table to each process**.
* Figure 9.9 shows the basic idea:

![](_attachments/Screenshot%202023-04-07%20at%2016.01.53.png)

* Notice that multiple VPs can be mapped to the same shared physical page. 
* VM simplifies the way that memory is used and managed in a system. In particular:

> * **Linking**: The virtual address space for each process has the same basic format. This simplifies the design and implementation of linkers.
> * **Loading**: It's easy to load executable and shared object files into memory. To load an object file into a newly created process, the Linux loader allocates virtual pages for the code and data segments, marks them as invalid (i.e. not cached), and points their page table entries to the appropriate locations in the object file. The loader never actually copies any data from disk to memory. The data are paged in on demand by the VM system the first time each page is referenced.
>   This idea of mapping a set of contiguous virtual pages to an arbitrary location in an arbitrary file is called [Memory Mapping](Memory%20Mapping.md).[^fn1] [^fn2]
>  * **Sharing**: Separate address spaces provide a consistent mechanism for managing sharing between user processes and the OS itself. Sometimes, it's desirable for processes to share code and data. For example, every process must call the same OS kernel code. Rather than including separate copies of the kernel in each process, multiple processes cam share a single copy of this code by mapping the appropriate VPs in different processes to the same physical pages. This describes Figure 9.9 (above).
>  * **Memory Allocation**: VM provides a simple mechanism for allocating additional memory to processes. When a process requests additional heap space (e.g. with `malloc`), the OS allocates some number of contiguous VM pages and maps them to physical pages. Because of the way page tables work, there is no need for the OS to locate the same number of contiguous pages of physical memory. They can be scattered randomly.


[^fn1]: I think that *memory mapping* is a pretty general term. It's just a technique that allows a program to access data from an external file (on disk) as if it were part of its own memory.
[^fn2]: [Memory-mapped I/O](Storage%20Technologies#^e55d6d) is another application of memory mapping. Instead of mapping a file to the VM space, it maps the registers of I/O devices to the VM space.