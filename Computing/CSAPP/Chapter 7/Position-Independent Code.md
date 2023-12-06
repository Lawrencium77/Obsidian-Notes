A key purpose of shared libraries is to allow multiple running processes to share the same library code in memory. We memory-map the shared objects into a [specific region of VM used for this purpose](Loading%20Executable%20Object%20Files.md). 

This requires that we compile the code segments of shared modules such that they can be loaded anywhere in memory without having to be modified by the linker. Once this is achieved, a single copy of a shared module's code segment can be shared by an unlimited number of processes. (Of course, each process will still get its own copy of the read/write data segment.)

Code that can be loaded without any relocations is called **position-independent code (PIC)**. 

References to local symbols (i.e., those in the *same* executable object module) require no special treatment to be PIC; they can be compiled using PC-relative addressing. 

However, references to global symbols defined outside the shared object require more careful treatment; the referenced symbol could exist at many different memory addresses, depending on how the various different shared objects are loaded. 

## PIC Data References
To be clear, the challenge we face is that:

* References to external symbols need to be resolved. The address of these symbols will vary between different processes that load the same shared object.
* But we need to use exactly the same shared object code across multiple processes. 

The solution exploits the following fact: no matter where we load an object module in memory, the data segment is always the same distance from the code segment. Thus, the ***distance*** between any instruction in the code segment and any variable in the data segment is a run-time constant.

The solution used is called a **global offset table (GOT)**. This is added at the beginning of the data segment. The GOT contains an 8-byte entry for each global data object (procedure or global variable) that is referenced by the object module. The compiler also generates a relocation record for each entry in the GOT. At load time, the dynamic linker relocates each GOT entry so that it contains the absolute address of the object. Each object module that references global objects has its own GOT.

Figure 7.18 shows the GOT from our example `libvector.so`:

![](_attachments/Screenshot%202023-10-29%20at%2017.35.16.png)

The `addvec` routine loads the address of the global variable `addcnt` in memory indirectly via `GOT[3]` and then increments `addcnt` in memory. They key idea here is that the offset in the PC-relative reference to `GOT[3]` is a runtime constant. 

This example isn't a great one: since `addcnt` is defined by `libvector.so`, the compiler could have exploited the constant distance between the code and data segments by generating a direct PC-relative reference to `addcnt` and adding a relocation for the linker to resolve when it builds the shared module. However, if `addcnt` were defined by another shared module, then the indirect access through the GOT would be necessary. 
## PIC Function Calls
Suppose a shared object calls a function that is defined by another shared object. The compiler has no way of predicting the run-time address of the function, but we need a solution that guarantees PIC.

GNU compilation systems solve this problem using an interesting technique called **lazy binding**, that defers the binding of each procedure address until the **first time** that it's called.

The motivation for lazy binding is that a typical application program will call only a handful of the thousands of functions exported by a shared library. By deferring the resolution of a function's address until it's actually called, the dynamic linker can avoid hundreds or thousands of unnecessary relocations. There is a nontrivial run-time overhead the first time the function is called, but each call thereafter costs only a single instruction and a memory reference for the indirection. 

Lazy binding is implemented with a compact yet somewhat complex interaction between two data structures: the GOT and the **procedure linkage table (PLT)**. If an object module calls any functions that are defined in shared libraries, then it has its own GOT and PLT. The GOT is part of the data segment. The PLT is part of the code segment. 

> [!INFO]
> I chose not to cover the details of the interaction between the GOT and PLT. I can come back round to it if I fancy.