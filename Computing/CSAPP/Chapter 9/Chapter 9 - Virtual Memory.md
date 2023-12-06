Processes in a system share the same CPU and main memory (MM).
Sharing MM poses some challenges. If too many processes need too much memory, then some cannot run. MM is also vulnerable to corruption - one process might inadvertently write to the memory used by another process.

To manage memory effectively, systems provide an abstraction of main memory called [virtual memory](The%20Operating%20System%20Manages%20the%20Hardware#^90ee75). This is a combination of hardware exceptions, address translation, MM, disk files, and kernel software. It provides each process with a large, uniform, private address space.

This chapter's contents are:

* [Physical and Virtual Addressing](Physical%20and%20Virtual%20Addressing.md)
* [Address Spaces](Address%20Spaces.md)
* [VM as a Tool for Caching](VM%20as%20a%20Tool%20for%20Caching.md)
* [VM as a Tool for Memory Management](VM%20as%20a%20Tool%20for%20Memory%20Management.md)
* [VM as a Tool for Memory Protection](VM%20as%20a%20Tool%20for%20Memory%20Protection.md)
* [Address Translation](Address%20Translation.md)
* [Case Study - The Intel Core i7 and Linux Memory System](Case%20Study%20-%20The%20Intel%20Core%20i7%20and%20Linux%20Memory%20System.md)
* [Memory Mapping](Memory%20Mapping.md)
* [Dynamic Memory Allocation](Dynamic%20Memory%20Allocation.md)
* [Garbage Collection (CSAPP)](Garbage%20Collection%20(CSAPP).md)
* [Common Memory-Related Bugs in C Programs](Common%20Memory-Related%20Bugs%20in%20C%20Programs.md)
