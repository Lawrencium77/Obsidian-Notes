* An address space is an ordered set of nonnegative integer addresses:

$$\{0,1,2,\dots\}$$

* If the integers are consecutive, we call it a **linear address space**. We always assume linear address spaces.
* In a system with virtual memory, the CPU generates virtual addresses an address space called the **virtual address space**.
* The size of an address space is determined by the number of bits needed to represent the largest address. A virtual address space with $N=2^n$ addresses is called an $n$-bit address space. $n$ is the [word size](Information%20Storage#^3e73dc) of the program.
* Modern systems typically support either 32 or 64-bit virtual address spaces:

$$\{0,1,2,\dots,N-1\}$$

* A system also has a **physical address space** that corresponds to the $M$ bytes of physical memory in the system:

$$\{0,1,2,\dots,M-1\}$$





