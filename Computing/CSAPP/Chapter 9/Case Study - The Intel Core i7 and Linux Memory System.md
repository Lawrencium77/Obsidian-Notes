This section concludes our discussion of virtual memory mechanisms by studying an Intel Core i7 running Linux.
The first part describes address translation in the i7 core. I skipped making notes on this.
The second describes the Linux virtual memory system, which I found more useful.

```toc
```

## Linux Virtual Memory System
* Linux maintains a separate virtual address space for each process in the following form:

![](_attachments/Screenshot%202023-05-05%20at%2014.07.11.png)

* This picture regularly occurs throughout CSAPP, including in the [first chapter](../Chapter%201/The%20Operating%20System%20Manages%20the%20Hardware.md).
* We see code, data, heap, shared library, and stack segments.
* Understanding [Address Translation](Address%20Translation.md) allows us to better understand the kernel virtual memory that lies above the user stack.
* Some regions of kernel virtual memory are mapped to physical pages that are shared by all processes. For instance, each process shares the kernel's code and  data.
* Linux also maps a set of contiguous virtual pages (equal in size to the total amount of DRAM in the system) to the corresponding set of contiguous physical pages.
* This provides the kernel with a way to access specific locations in physical memory - for instance, when it needs to access page tables.

> [!QUESTION]
> I don't quite get that last two bullet points.

* Other regions of the kernel virtual memory contain data that differ for each process. Examples include page tables, the stack that the kernel uses when executing code in the context of the current process, and data structures that keep track of the organisation of the virtual address space.

### Linux Virtual Memory Areas
* Linux organises virtual memory as a collection of **areas** (aka **segments**).
* An area is a contiguous chunk of allocated virtual memory whose pages are related in some way.
* The code segment, data segment, heap, and stack are all distinct areas. ^371c5c
* Each existing virtual page is contained in some area, and any virtual page that is not part of some area doesn't exist and can't be referenced by the process.
* The notion of an area allows the virtual address space to have **gaps**.
* Figure 9.27 shows the kernel data structures that keep track of the virtual memory areas in a process:

![](_attachments/Screenshot%202023-05-05%20at%2014.23.14.png)

* I've seen very similar diagrams in the [TDLP](../../Linux/The%20Linux%20Kernel/0%20-Intro.md) book that I studied! See [here](4%20-%20Processes#Files).
* Each `task_struct` contains an entry that points to an `mm_struct`. This `mm_struct` characterises the current state of virtual memory. It has two fields of interest to us:
	* `pgd` - points to the base of the level 1 table
	* `mmap` - points to a list of `vm_area_structs`, each of which characterises an area of the virtual address space.
* When the kernel runs a process, it stores `pgd` in a control register.
* An area struct contains the following fields:
	* `vm_start` - points to the beginning of the area
	* `vm_end` - points to the end of the area
	* `vm_flags` - describes (amongst other things) whether the area's pages are shared with other processes
	* `vm_next` - points to the next area struct

### Linux Page Fault Exception Handling
Suppose the MMU triggers a [page fault](VM%20as%20a%20Tool%20for%20Caching#Page%20Faults) while trying to translate a virtual address, $A$. The exception results in a transfer of control to the kernel's page fault handler. This then performs the following steps:

1. Is $A$ legal - does it lie within an area defined by some struct? To answer this, the fault handler searches the list of area structs, comparing $A$ with each `vm_start`   and `vm_end`. If the instruction is not legal, it triggers a [segfault](VM%20as%20a%20Tool%20for%20Memory%20Protection#^30cbf1). 
	In practice, Linux actually does a [tree search](https://en.wikipedia.org/wiki/Search_tree) over the list of area structs, using some fields we have not shown. 
2. Is the attempted memory access legal? I.e. does the process have permission to read, write, or execute the pages in this area? If not, the fault handler triggers a protection exception, which terminates the process. 
3. At this point, the kernel knows the page fault resulted from a legal operation on a legal memory address. It handles the fault by selecting a victim page, and swapping in a new page, updating the page table in the process.

> [!INFO]
> It seems there are multiple possible causes for a segfault. Examples include attempting to access a memory address outside of the process's address space, and violating permissions. These correspond to scenarios 1 and 2 above. See [Wikipedia](https://en.wikipedia.org/wiki/Segmentation_fault) for more detail.

These three scenarios are described in Figure 9.28:

![](_attachments/Screenshot%202023-05-05%20at%2016.54.07.png)

