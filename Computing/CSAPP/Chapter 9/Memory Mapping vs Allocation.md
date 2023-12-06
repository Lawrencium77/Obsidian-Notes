What are the exact differences and similarities between [Memory Mapping](Memory%20Mapping.md) and [Memory Allocation](Dynamic%20Memory%20Allocation.md)?

I'm not 100% certain but my read of CSAPP [Chapter 9 - Virtual Memory](Chapter%209%20-%20Virtual%20Memory.md) is as follows:

> * **Both** are mechanisms that a process can use to acquire additional virtual memory.
> * Memory **Allocation** is simply about reserving memory for use by a program. This happens on the heap, and all allocated pages are demand-zero.
> * Memory **Mapping** is about setting up a direct correspondence between memory addresses and physical memory. But we can also create a mapping to an anonymous file (i.e. demand-zero pages), in which case it seems very similar to memory allocation.