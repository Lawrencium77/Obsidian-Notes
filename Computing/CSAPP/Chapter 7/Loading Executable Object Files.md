To run an executable object file `prog`, we simply do:

```
linux> ./prog
```

Since `prog` doesn't correspond to a built-in shell command, the shell assumes `prog` is an executable object file, which it runs for us by invoking some memory-resident operating system code called the [loader](Compiler%20Drivers#^727a6f). Any Linux program can invoke the loader by calling `execve`. This does a bunch of stuff (I've covered this in other notes), including memory mapping the code and data in the executable object file on disk, and runs the program by jumping to its first instruction. 

Every running Linux program has a run-time memory image similar to Figure 7.15:

![](_attachments/Screenshot%202023-10-29%20at%2011.03.55.png)

On Linux x86-64 systems, the code segment starts at address `0x400000`, and is followed by the data segment. The run-time [heap](../Chapter%209/Dynamic%20Memory%20Allocation.md) follows the data segment, and grows upwards via calls to `malloc`. This is followed by a region that's reserved for shared modules. The user stack starts at the largest legal user address and grows down, towards smaller addresses. The region above the stack is reserved for code and data in the OS kernel.

For simplicity, we've shown the heap, data, and code segments as abutting each other. In practice, there is a gap between the code and data segments due to the [alignment requirement](Executable%20Object%20Files#^0ef570) on the `.data` segment. Also, the linker uses [stack randomisation](Combining%20Control%20and%20Data%20in%20Machine-Level%20Programs#Stack%20Randomisation). 

When the loader runs, it creates a memory image similar to the one shown in Figure 7.15. Guided by the [program header table](Executable%20Object%20Files.md), it copies chunks of the executable object file into the code and data segments. Next, the loader sets the PC to be the program's entry point. 

This entry point is always the address of the `_start` function, which is defined in the system object file `ctrl.o` and is the same for all C programs. The `_start` function calls the ***system startup function***, `__libc_start_main`, which initialises the execution environment, calls the user-level `main` function, handles its return value, and if necessary returns control to the kernel.

