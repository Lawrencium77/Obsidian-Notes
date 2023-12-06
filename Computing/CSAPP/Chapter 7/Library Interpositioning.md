> [!INFO]
> I found this sub-chapter a little boring/confusing. So I didn't really make any Ankis on it.



Linux linkers support a powerful technique called [library interpositioning](https://stackoverflow.com/questions/65275140/library-interpositioning), that allows you to intercept calls to shared library functions and execute your own code instead.

Here's the basic idea: given some **target function**, you create a **wrapper function** whose prototype is identical to the target function. Using some interpositioning mechanism, you then trick the system into calling the wrapper function instead of the target function.

Interpositioning can happen a compile time, link time, or run time. To explore these different mechanisms, we'll use the example program in Figure 7.20(a):

![](_attachments/Screenshot%202023-11-12%20at%2013.46.25.png)

This calls `malloc` to allocate a 32-byte block from the heap. It then frees this block.

## Compile-Time Interpositioning
This is quite straightforward. We tell the preprocessor to interpose in a header file:

![](_attachments/Screenshot%202023-11-12%20at%2013.48.48.png)

with corresponding source code in `mymalloc.c`. This is then compiled as:

```
linux> gcc -DCOMPILETIME -c mymalloc.c
linux> gcc -I. -o intc int.c mymalloc.c
```

The `-I.` argument tells the C preprocessor to look for `malloc.h` in the current directory before looking in the usual system directories. 

## Link-Time Interpositioning
The Linux static linker supports link-time interpositioning with the `--wrap f` flag. This tells the linker to resolve references to `f` as `__wrap_f`, and to resolve references to symbol `__real_f` as `f`.

I skipped the rest of this; it's not too interesting.

## Run-Time Interpositioning
Compile-time interpositioning requires access to a program's source files. Link-time interpositioning requires access to its relocatable object files. However, run-time interpositioning requires access only to the executable object file. This mechanism is based on the dynamic linker's `LD_PRELOAD` environment variable.

If `LD_PRELOAD` is set to a list of shared library pathnames, then when you load and execute a program, the dynamic linker will search the `LD_PRELOAD` libraries first, before any other shared libraries, when it resolves undefined references. With this mechanism, you can interpose on any function in any shared library.



