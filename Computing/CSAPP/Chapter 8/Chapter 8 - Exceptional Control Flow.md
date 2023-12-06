As long as a processor is running, the program counter assumes a sequence of values:

$$a_0,a_1,\dots,a_{n-1}$$
where each $a_k$ is the address of some instruction $I_k$. Each transition from $a_k$ to $a_{k+1}$ is called a **control transfer**. A sequence of control transfers is called the **control flow** of a processor.

The simplest kind of flow is a "smooth" sequence where each $I_k$ and $I_{k+1}$ are adjacent in memory. Typically, abrupt changes to this smooth flow are caused by program instructions such as jumps, calls, and returns.

But systems may also be able to react to changes in system state that are not necessarily related to the execution of the program. For example, a hardware timer goes off at regular intervals and must be dealt with. Packets arrive at the network adapter and must be stored in memory. Etc.

Modern systems react to these situations by making abrupt changes in the control flow. In general, we refer to these abrupt changes as **exception control flow (ECF)**.  ^870052

ECF occurs at all levels of a computer system. For example, at the hardware level, events detected by the hardware trigger abrupt control transfers to exception handlers. At the OS level, the kernel transfers control from one user process to another via context switches. An individual program can react to errors by making nonlocal jumps to arbitrary locations in other functions.

The contents of this chapter are as follows:

* [Exceptions](Exceptions.md)
* [Processes](Processes.md)
* [System Call Error Handling](System%20Call%20Error%20Handling.md)
* [Process Control](Process%20Control.md)
* [Signals](Signals.md)
* [Nonlocal Jumps](Nonlocal%20Jumps.md)
* [Tools for Manipulating Processes](Tools%20for%20Manipulating%20Processes.md)

