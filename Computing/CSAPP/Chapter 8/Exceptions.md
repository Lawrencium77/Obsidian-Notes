```toc
```

An **exception** is an abrupt change to the control flow in response to some change in the processor's state. Figure 8.1 shows the basic idea:

![](_attachments/Screenshot%202023-06-05%20at%2015.59.27.png)

In this figure, the processor is executing some current instruction $I_{curr}$ when a significant change in the processor's **state** occurs. The state is encoded in various bits and signals inside the processor. The change in state is called an **event**.

The event might be directly related to the execution of the current instruction. For instance, a page fault may occur, or arithmetic overflow. It could also be unrelated, for instance when an I/O request completes.

In any case, when the processor detects that an event has occurred, it makes a procedure call [^fn1] (the **exception**), with a jump table (called an **exception table** - see below for more), to an OS subroutine (the **exception handler**) that is designed to handle this particular event. When the exception handler finishes processing, one of three things happens, depending on the type of event that caused the exception:

1. The handler returns control to $I_{curr}$
2. The handler returns control to $I_{next}$
3. The handler aborts the interrupted program

## Exception Handling
Exceptions can be hard to understand because handling them involves close cooperation between the hardware and software. It's easy to get confused about which component performs which task. Let's look at the division of work in more detail.

Each type of possible exception in a system is assigned a unique integer **exception number**. 

When the system is turned on, the OS allocates and initialises a jump table called an **exception table**, such that entry $k$ contains the address of the handler for exception $k$. Figure 8.2 shows this idea:

![](_attachments/Screenshot%202023-06-05%20at%2016.10.30.png)

At run time, the processor detects that an event has occurred and determines the corresponding exception number $k$. The processor then triggers the exception by making a procedure call, through entry $k$ of the exception table, to the corresponding handler. Figure 8.3 shows how the processor uses the exception table to form the address of the appropriate handler:

![](_attachments/Screenshot%202023-06-05%20at%2016.19.05.png)

The exception number is an index into the exception table, whose starting address is contained in a CPU register called the **exception table base register**.

Once the hardware triggers the exception, the rest of the work is done in software by the exception handler. After the handler has processed the event, it optionally returns to the interrupted program by executing a "return from interrupt" instruction. This pops the appropriate state back into the processor's control and data registers.


## Classes of Exceptions
Exceptions can be divided into four classes: **interrupts**, **traps**, **faults**, and **aborts**.
Figure 8.4 summarises these:

![](_attachments/Screenshot%202023-06-05%20at%2016.23.14.png)

### Interrupts
Interrupts occur *asynchronously* as a result of signals from I/O devices that are external to the processor. Hardware interrupts are asynchronous in the sense that they're not caused by the execution of any particular instruction. Exception handlers for hardware interrupts are often called **interrupt handlers**.

Figure 8.5 summarises the processing for an interrupt:

![](_attachments/Screenshot%202023-06-05%20at%2016.34.09.png)

I/O devices such as network adapters and disk controllers trigger interrupts by signalling a pin on the processor chip and placing onto the system bus the exception number that identifies the device that caused the interrupt.

After the current instruction finishes executing, the processor notices that the interrupt pin has gone high, reads the exception number for the system bus, and then calls the appropriate interrupt handler. When the handler returns, it returns control to the next instruction (i.e. the instruction that would have followed the current instruction in the control flow had the interrupt not occurred).

The remaining classes of exceptions (traps, faults, and aborts) occur **synchronously**, as a result of executing the current instruction. We refer to this instruction as the **faulting instruction**.

### Traps and System Calls
Traps are *intentional* exceptions that occur as a result of executing an instruction. Like interrupt handlers, trap handlers return control to the next instruction. The most important use of traps is to provide an interface between user programs and the kernel, called a **system call**.

User programs often need to request services from the kernel, such as `read`, `fork`, and `execve`. To allow controlled access to such kernel services, processors provide a special `syscall` instruction that user programs can execute when they require some service.

Executing `syscall` causes a trap to an exception handler that decodes the argument and calls the appropriate kernel routine:

![](_attachments/Screenshot%202023-06-05%20at%2016.45.02.png)

From a programmer's perspective, a system call is identical to a regular function call. However, their implementations are different. Regular functions run in *user mode*, whereas syscalls run in *kernel mode*.

Note that syscalls execute within the context of the calling process; in other words, no [context switch](Processes#Context%20Switches) is required.

### Faults
Faults result from error conditions that a handler might be able to correct. When a fault occurs, the processor transfers control to the fault handler. If the handler is able to correct the error condition, it returns control to the faulting instruction, thereby re-executing it. Otherwise, the handler returns control to an `abort` routine in the kernel that terminates the application program that caused the fault. 

Figure 8.7 summarises this:

![](_attachments/Screenshot%202023-06-05%20at%2016.49.22.png)

A classic example is the [page fault](Case%20Study%20-%20The%20Intel%20Core%20i7%20and%20Linux%20Memory%20System#Linux%20Page%20Fault%20Exception%20Handling) exception, which occurs when an instruction references a virtual address whose corresponding page is not resident in memory. 

### Aborts
Aborts result from unrecoverable fatal errors. Abort handlers never return control to the application program. As shown in Figure 8.8, the handler returns control to an `abort` routine that terminates the application program:

![](_attachments/Screenshot%202023-06-05%20at%2016.52.29.png)


## Exceptions in Linux/x86-64 Systems
To make things more concrete, let's look at some of the exceptions defined for `x86` systems:

#### Linux/x86-64 Faults and Aborts
* **Divide Error**: Occurs when an application attempts to divide by zero, or when the result is too big to represent. Unix doesn't attempt to recover from divide errors, instead opting to abort the program. 
* **General Protection Fault**: This occurs for many reasons, usually because a program references an undefined area of [virtual memory](../Chapter%209/Chapter%209%20-%20Virtual%20Memory.md), or because the program attempts to write to a read-only text segment. Linux shells typically report this as a [segfault](VM%20as%20a%20Tool%20for%20Memory%20Protection#^30cbf1).
* [Page Fault](Address%20Translation#^d0e7a7): An example of an exception where the faulting instruction is restarted. The handler swaps the appropriate page into DRAM and then restarts the faulting instruction.

#### Linux/x86-64 System Calls
Linux provides hundreds of system calls that programs use when they want to request services from the kernel. Figure 8.10 lists some popular examples:

![](_attachments/Screenshot%202023-07-16%20at%2010.48.47.png)

C programs can invoke any system call directly by using the `syscall` function. However, this is rarely necessary in practice. The C standard library provides a set of wrapper functions for most system calls. These package up the arguments, trap to the kernel with the appropriate system call instruction, and then pass the return status of the syscall back to the calling program.
As an example, here is a version of a *Hello World* program that uses the `write` system-level function instead of `printf`:

```C
int main()
{
	write(1, "hello, world\n", 13);
	_exit(0)
}
```

The first argument to `write` sends the output to `stdout`. The second argument is the bytes to write, and the third gives the number of bytes to write. 
The idea of this example is that `printf` would often cause the `write` syscall to eventually be invoked. But we can use `write` directly if we wish.

For the rest of these notes, we refer to system calls and their associated wrapper function as **system-level functions**.













[^fn1]: A "procedure" is a set of instructions grouped together to perform a specific task. A "procedure call" is an instruction in a program that requests some procedure to be executed.


