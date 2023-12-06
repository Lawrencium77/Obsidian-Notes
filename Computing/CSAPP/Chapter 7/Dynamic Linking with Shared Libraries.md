[Static libraries](Static%20Linking.md) address many of the issues associated with making large collections of related functions available to a program. However, static libraries still have some significant disadvantages:

* If a library is updated, the application programmer must relink their programs against the updated library.
* Almost every C program uses standard I/O functions such as `printf`. At run time, the code for these functions is duplicated in the text segment of each running process. On a typical system that's running hundreds of processes, this can be a significant waste of memory.

[Shared libraries](https://en.wikipedia.org/wiki/Shared_library) are modern innovations that address these disadvantages. A shared library is an object that, at either run time or load time, can be loaded at an arbitrary memory address and linked with a program in memory. This process is called **dynamic linking** and is performed by a **dynamic linker**. Shared libraries are also called **shared objects**, and on Linux systems have the `.so` suffix. 

Shared libraries are "shared" in two ways:

1. In any given file system, there is exactly one `.so` file for a particular library. The contents of this `.so` file are shared by all executable object files that reference it. This is unlike static libraries, which are copied and embedded into the executables that reference them.
2. A single copy of the `.text` section of a shared library in memory can be shared by different running processes. 

## How Dynamic Linking Works

> [!INFO]
> These notes are based on CSAPP, but I added a little extra to them.

First, let's look at the commands that we run to perform dynamic linking. Figure 7.16 summarises the dynamic linking process for the example program in [Figure 7.7](Symbol%20Resolution.md):

![](_attachments/Screenshot%202023-10-29%20at%2011.35.32.png)

To build a shared library `libvector.so` of our example vector routines in Figure 7.6, we'd do:

```
linux> gcc -shared -fpic -o libvector.so addvec.c multvec.c
```

Once we've created the library, we would then link it into our example program in Figure 7.7:

```
linux> gcc -o prog2l main2.c ./libvector.so
```

This creates a partially linked executable `prog2l` in a form that can be linked with `libvector.so` at run time. 

The basic idea is that we do some of linking statically, and then complete the linking process dynamically when the program is loaded. It's important to realise that none of the code or data sections from `libvector.so` are actually copied into the executable `prog2l`. Instead of resolving all symbols at link time, as you would do for [Static Linking](Static%20Linking.md), the linker copies some relocation and symbol table information, and keeps a record of unresolved symbols in the **dynamic symbol table** within the resulting executable. This contains information allows references to code and data in `libvector.so` to be resolved at load time. The executable also contains a section called `.interp`, which specifies the dynamic linker to be used a load time, which is itself a shared object (e.g. `ld-linux.so` on Linux systems).

Next, the loader loads and runs `prog2l`. It notices that `prog2l` contains a `.interp` section. Instead of passing control to the application, the loader loads and runs the dynamic linker, which finishes the linking task by performing the following steps:

* Relocating the text and data of `libc.so` and `libvector.so` into [some segment of VM](Loading%20Executable%20Object%20Files.md). Both are memory-mapped, but the data segment is set to be [copy-on-write](Memory%20Mapping#^990957), ensuring that each process has a data segment that it can modify.
* Resolving any references in `prog2l` to symbols defined by `libc.so` and `libvector.so`. To do so, it uses a procedure similar to [traditional linking process](symbol%20resolution#Linking%20with%20Static%20Libraries).
* Relocating these references in `prog2l`.

Finally, the dynamic linker passes control to the application. The locations of the shared libraries are fixed for the execution of the program.



