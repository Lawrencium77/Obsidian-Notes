I've [other notes](../../Linux/The%20Linux%20Kernel/4%20-%20Processes.md) on this topic in the past. So I just made notes on the parts that were new to me.

## Private Address Space
I thought it useful to copy this diagram, showing the organisation of the address space for an x86-64 Linux process:

![](_attachments/Screenshot%202023-07-16%20at%2011.06.35.png)

The bottom portion is reserved for the use program, with the usual code, data, heap, and stack segments. The top portion is reserved for the kernel. This contains the code, data, and stack that the kernel uses when it executes instructions on behalf of a process.

## User and Kernel Modes
The processor must provide a mechanism that restricts the instructions an application can execute, and the addresses that it can access. They typically do this with a **mode bit** in some control register. When the mode bit is set, the process is running in **kernel mode**; any process running in kernel mode can execute any instruction and access any memory location.

When the mode bit is not set, the process runs in **user mode**. A process running in user mode cannot execute certain instructions, such as those that halt the processor or initiate an I/O operation. Nor is it allowed to reference an address in the kernel area of the address space.

A process running application code is initially in user mode. The only way for this to change is via an exception. The exception handler runs in kernel mode.

The `/proc` filesystem provides *controlled* access to kernel information. The contents of many kernel data structures are exported into `/proc` as a hierarchy of text files that can be read by user programs. By offering a read-only interface, `/proc` prevents unauthorised modifications while still allowing visibility into the kernel state.

## Context Switches
The OS kernel implements multitasking using a higher-level form of exceptional control flow called the **context switch**. This is built on top of the lower-level exception mechanisms discussed [previously](Exceptions.md).

The context is the state that the kernel needs to restart a preempted process.

It is worth studying the anatomy of a context switch in some detail. Consider Figure 8.14:

![](_attachments/Screenshot%202023-07-16%20at%2011.25.13.png)

Initially, `A` is running in user model until it traps the kernel by executing the `read` system call. The trap handler in the kernel requests a DMA transfer from the disk controller and arranges for the disk to interrupt the processor once the transfer is complete.

The disk will take some time to fetch the data, so instead of waiting and doing nothing in the interim, the kernel performs a context switch from process `A` to `B`. 

Note that, before the switch, the kernel is executing instructions on behalf of process `A`; there is no separate, pre-existing kernel process. During the first part of the switch, the kernel is executing instructions in kernel mode on behalf of process `A`. At some point, it begins executing instructions (still in kernel mode) on behalf of process `B`. After the switch, process `B` is then executing in user mode.





