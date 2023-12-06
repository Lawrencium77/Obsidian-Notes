Up to this point, we have discussed the scenario in which the dynamic linker loads and links shared libraries when an application is loaded, just before it executes. However, it's possible for an application to request the dynamic linker to load and link arbitrary shared objects while the application is running.

Linux provides a simple interface to the dynamic linker that allows application programs to load and link shared libraries at run time:

![](_attachments/Screenshot%202023-10-29%20at%2011.56.59.png)

The `dlopen` function loads and linked the shared library `filename`. The external symbols in `filename` are resolved using libraries previously opened with the `RTLF_GLOBAL` flag.

The `dlsym` function takes a `handle` to a previously opened shared library and a `symbol` name. It returns the address of the symbol if it exists, or `NULL` otherwise:

![](_attachments/Screenshot%202023-11-04%20at%2015.29.58.png)

Finally, the `dlclose` function unloads the shared library if no other libraries are still using it:

![](_attachments/Screenshot%202023-11-04%20at%2015.31.26.png)

Figure 7.17 shows an example program that dynamically links against a shared library at run time. The key point is that the `addvec` routine can only be invoked once we call `dlopen`:

![](_attachments/Screenshot%202023-11-04%20at%2015.32.28.png)

> [!INFO] 
> The information below summarises some extra reading I did on run-time vs load-time dynamic linking.

## Load-Time vs Run-Time Dynamic Linking
The key idea of [dynamic linking](Dynamic%20Linking%20with%20Shared%20Libraries.md) is that it uses a shared library that is not compiled into the program. Instead, the shared library can be loaded at either load time or run time. The following describes the two scenarios. 

### Load-Time Dynamic Linking
This is the most common form. This description repeats that contained in [Dynamic Linking with Shared Libraries](Dynamic%20Linking%20with%20Shared%20Libraries.md).

1. **Compile Time**: At compile time, the program is compiled with references to symbols that are defined in the shared library. 
2. **Load Time**: The OS linker/loader performs symbol resolution, and relocation. The program is then able to run.

### Run-Time Dynamic Linking

1. **Compile Time**: The program is compiled without a shared library - no linking against the shared library is done at this stage. The program contains code to explicitly request loading a library and look up symbols at run time.
2. **Run Time**: The program itself contains code that calls `dlopen` (Unix) to load the shared library into memory, and `dlsym` to find the addresses of the symbols it needs to call. This allows the program to decide when to load the library, which can be long after the program has started running.

### CMake: STATIC vs SHARED vs MODULE
This relates to the concept of `STATIC` vs `SHARED` vs `MODULE` when [making a library](CMake#Making%20a%20Library) in CMake. To summarise:

* `STATIC` - creates a static library.
* `SHARED` - creates a shared library, used for load-time dynamic linking.
* `MODULE` - creates a shared library, used for run-time dynamic linking.




