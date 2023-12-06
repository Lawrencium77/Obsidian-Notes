Linux initialises the contents of virtual memory by associating it with an **object** on disk. This is called **memory mapping**. Areas can be mapped to **two** types of object:

1. **Regular file in the Linux file system**: A contiguous section of a regular disk file, such as some executable object file. The file section is divided into page-size pieces, with each piece containing the contents of a virtual page.
2. **Anonymous file**: These are created by the kernel. They contain all binary zeros. The first time the CPU touches[^fn1] a virtual page in such an area, the kernel finds a victim page in DRAM, swaps out the victim page, overwrites it page with binary zeros, and updates the page table. Notice that **no data are transferred from disk to memory**. For this reason, pages in areas that are mapped to anonymous files are sometimes called **demand-zero pages**. ^b05890

Once a virtual page is initialised, the kernel maintains a dedicated space on disk called a **swap file**. This acts as temporary storage area for pages that have been evicted. When an evicted page needs to be accessed again, the OS will read the updated version from the swap file. ^05c2fd

To understand this better, suppose we evict a page from DRAM. It is swapped into the swap file. There are now two copies of the page on disk: the original (in its original location), and updated version (in the swap file). The original page is not updated, since it serves as a reference for the initial contents of the page (e.g. when used by multiple processes).

When a page is swapped back from the swap space to DRAM, the page still exists in the swap space, but the OS typically marks it as "available" for future use.

An important point to note is that the size of the swap space limits the total number of pages that can be allocated by **all processes** running on a system. When the swap space is full, no more virtual pages can be allocated. This is because the OS needs to ensure it has space to store evicted pages from RAM.

### Shared Objects Revisited
* As we've seen, the OS provides each process with it own virtual address space.
* However, many processes have identical read-only code areas. For example, each process that runs `bash` has the same code [area](Case%20Study%20-%20The%20Intel%20Core%20i7%20and%20Linux%20Memory%20System#^371c5c).
* It would be wasteful to maintain duplicate copies of commonly used code in DRAM. Memory mapping provides a mechanism for sharing objects between processes.
* An object can be mapped into an area of virtual memory as either a **shared object** or a **private object**. 
* If a process maps a shared object into its virtual address space, then any writes the process makes are visible to other processes that share it.
* The changes are also reflected in the original object on disk.
* This is shown in the following diagram:

![](_attachments/Screenshot%202023-05-05%20at%2017.23.47.png)

* In contrast, changes made to **private object** are not visible to other processes, and are **not** reflected back to the object on disk. 
* A virtual memory area into which a shared object is mapped is called a **shared area**. 
* Similarly for a **private area**.
* Suppose that process 1 maps a shared object into an area of its VM, and that process 2 does the same (Figure 9.29 (b)).
* Since each object has a unique filename, the kernel can determine that process 1 has already mapped this object and can point the page table entries in process 2 to the appropriate physical pages.
* Private objects are mapped into virtual memory using a technique called **copy-on-write**. A private object begins life in exactly the same way as a shared object, with only one copy in physical memory (Figure 9.30a).

![](_attachments/Screenshot%202023-05-05%20at%2020.59.19.png)

* For each process that maps the private object, the page table entries for the corresponding private area are flagged as **read-only**, and the area struct is flagged as **private-copy-on-write**. 
* So long as neither process attempts to write to its respective area, they continue to share a single copy of the object.
* But once a process attempts to write to some page in the private area, the write triggers a protection fault.
* When the fault handler notices the protection exception was caused by the process of trying to write to a page in a private copy-on-write area, it creates a new copy of the page in physical memory, and updates the page table entry to point to the new copy. It then restores write permissions to the page.
* This is shown in Figure 9.30b.
* Copy-on-write defers copying of pages in private objects until the last possible moment, thus making most efficient use of physical memory. ^990957

### The `fork` Function Revisited
* We can now understand how `fork` creates a new process with its own independent virtual address space.
* When `fork` is called by the **current process**, the kernel creates various data structures for the **new process** and assigns it a unique PID. 
* To create the VM for the new process, it creates exact copies of the current process' `mm_struct`, area structs, and page tables.
* It flags each page in both processes as **read-only**, and each area struct in both processes as private **copy-on-write**.
* When `fork` returns in the new process, the new process now has an exact copy of the virtual memory as it existed when `fork` was called. When either of the processes performs any subsequent writes, the copy-on-write mechanism creates new pages, thus preserving the abstraction of a private address space for each process.

### The `execve` Function Revisited
* VM and Memory Mapping are also important in loading programs into memory.
* Suppose the program running the current process makes the following call:

```C
execve("a.out", NULL, NULL)
```

* The `execve` function loads and runs the program contained in the executable object file `a.out` within the current process, effectively replacing the current program with `a.out`.
* Loading and running `a.out` requires the following steps:

1. **Deleting existing user areas**: Delete the existing area structs in the user portion of the current processes's virtual address space.
2. **Map private areas**: Create new area structs for the code, data, bss, and stack areas of the new program. All of these new areas are private copy-on-write. The code and data areas are mapped to the `.text` and `.data` sections of the `a.out` file. The bss area is demand-zero, mapped to an anonymous file whose size is contained in `a.out`. The stack and heap are also demand-zero, initially of zero length. Figure 9.31 summarises the mappings of the private areas:

![](_attachments/Screenshot%202023-05-05%20at%2021.15.22.png)

3. **Map shared areas**: If `a.out` was linked with shared objects, such as the standard C library, then these objects are dynamically linked into the program, and then mapped into the shared region of the user's virtual address space.
4. **Set the program counter (PC)**: Set the program counter to point to the entry point in the code area.

The next time the process is scheduled, it begins execution from the entry point.

### User-Level Memory Mapping with the `mmap` Function
* Linux processes use `mmap` to create new areas of virtual memory and to map objects into these areas.
* The API to `mmap` is as follows:

```C
void *mmap(void *start, size_t length, int prot, int flags, int fd,         
		   off_t offset);
```
* This returns a pointer to the mapped area.
* `mmap` asks the kernel to create a new virtual memory area, preferably one that starts at address `start`, and to map a contiguous chunk of the object specified by file descriptor `fd` to the new area.
* The contiguous object chunk has a size of `length` bytes and starts at an offset of `offset` bytes from the beginning of the file.
* The `start` address is merely a hint, and is usually specified as `NULL`.
* Figure 9.32 depicts this:

![](_attachments/Screenshot%202023-05-05%20at%2021.22.56.png)

* The `prot` argument contains bits that describe the access permissions of the newly mapped virtual memory area.
* The `flags` argument contains bits that describe the type of the mapped object. If the `MAP_ANON` flag bit is set, then the backing store is an anonymous object and the corresponding virtual pages are demand-zero.
* `MAP_PRIVATE` indicates a private copy-on-write object, and `MAP_SHARED` indicates a shared object.
* The `munmap` function deletes regions of VM:

```C
void *munmap(void *start, size_t length);
```

* This deletes the area starting at virtual address `start` and consisting of the next `length` bytes. Subsequent references to the deleted region cause segfaults.



[^fn1]: "Touching" a page means to issue a virtual address that falls within that page's region of the address space.