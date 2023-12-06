* Main memory (MM) is organised as an array of $M$ contiguous byte-size cells.
* Each byte has a unique **physical address** (PA).
* The most natural way for a CPU to access MM would be with physical addresses. We call this approach **physical addressing**.
* This diagram shows an example in the context of a load instruction. It reads a 4-byte word, starting at address 4:

![](_attachments/Screenshot%202023-04-07%20at%2014.24.32.png)

* Modern processors instead use **virtual addressing**:

![](_attachments/Screenshot%202023-04-07%20at%2014.25.56.png)

* The CPU accesses MM by generating a **virtual address** (VA). 
* This is converted to the appropriate PA before being sent to MM.
* The task of converting a virtual address to a physical address is called **address translation**.
* Dedicated hardware on the CPU chip called the **Memory Management Unit** (MMU) translates virtual addresses on the fly.
* It does so with a lookup table stored in MM. Its contents are managed by the OS. See the stuff on [page tables](VM%20as%20a%20Tool%20for%20Caching#^d48eb6) for more.

