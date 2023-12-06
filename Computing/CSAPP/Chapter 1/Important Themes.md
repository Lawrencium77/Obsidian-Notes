This concludes our initial whirlwind tour of computer systems. An important idea to take away is that a system is more than just hardware. It is a collection of intertwined hardware and software that must cooperate in order to run application programs. 

To close the chapter, we highlight several important concepts that cut across all aspects of computer systems.

### Amdahl's Law
**Amdhal's Law** is a simple but insightful observation about the effectiveness of improving the performance of one part of a system. The main idea is that when we speed up one part of a system, the effect on the overall system performance depends on how significant this part was and how much it sped up.

Consider a system in which executing some application requires time $T_{old}$. Suppose some part of the system requires a fraction $\alpha$ of this time, and that we improve its performance by a factor $k$. The component initially required time $\alpha T_{old}$ and now requires time $\frac{\alpha T_{old}}{k}$. The overall execution time would be:

![](_attachments/Screenshot%202022-05-18%20at%2016.02.25.png)

Meaning the speedup $S = T_{old}/T_{new}$ is:

![](_attachments/Screenshot%202022-05-18%20at%2016.02.53.png)

This is illustrated below:

[![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ea/AmdahlsLaw.svg/400px-AmdahlsLaw.svg.png)](https://en.wikipedia.org/wiki/File:AmdahlsLaw.svg)
The major insight is that to significantly speed up the entire system, we must improve the speed of a very large fraction of the overall system.

### Concurrency and Parallelism
**Concurrency** refers to the general concept of a system with multiple, simultaneous activities, and **parallelism** refers to the use of concurrency to make a system run faster. 

We highlight three levels here, working from the highest to lowest level in the system hierarchy.

**Thread-Level Concurrency**

We are able to devise systems where multiple programs execute at the same time, leading to *concurrency*. With threads, we can even have multiple control flows executing within a single process. Support for concurrent execution has been found in computer systems since the advent of **time-sharing** in the early 1960s. This concurrent execution is only *simulated*, by having a single computer rapidly switch among its executing processors. This configuration is a *uniprocessor system*.

When we construct a system with multiple processors all under the control of a single OS kernel, we have a *multiprocessor system*. Such systems have been available for large-scale computing since the 1980s, but have more recently become commonplace with the advent of *multi-core* processors and **hyperthreading**. Figure 1.16 shows a taxonomy of these different processor types.

![](_attachments/Screenshot%202022-05-19%20at%2008.13.48.png)

Multi-core processors have several CPUs integrated onto a single IC. Figure 1.17 shows the organisation of a typical multi-core processor. Each chip has 4 cores, each with its own L1 and L2 caches, and with each L1 cache split into two parts - one to hold recently fetched instructions, and one to hold data. The cores share higher levels of cache as well as the interface to main memory.

![](_attachments/Screenshot%202022-05-19%20at%2008.14.55.png)

Hyperthreading, sometimes called *simultaneous multi-threading*, is a technique that allows a single CPU to execute multiple flows of control. It involves having multiple copies of some of the CPU hardware, such as PCs and register files, while having only single copies of other parts of the hardware, such as the ALU. Whereas a conventional processor needs 20,000 clock cycles to shift between different threads, a hyperthreaded processor decides which of its threads to execute on a cycle-by-cycle basis. It enables the CPU to take better advantage of its processing resources. For example, if one thread must wait for some data to be loaded into cache, the CPU can proceed with the execution of a different thread.

**Instruction-Level Parallelism** ^12c8c3

At a much lower level of abstraction, modern processors can execute multiple instructions at one time, a property known as **instruction-level parallelism**. For example, early processors required multiple clock cycles to execute a single instruction. More recent processors can sustain execution rates of 2-4 instructions per clock cycle. Any given instruction requires much longer from start to finish (maybe 20 cycles or more), but the processor uses a number of clever tricks to process as many as 100 instructions at a time. 

In chapter 4, we will explore the use of **pipelining**, where the actions required to execute an instruction are partitioned into different steps and the processor hardware is organised into a series of stages, each performing one of these steps. The stages can then operate in parallel.

Processors that can sustain execution rates faster than 1 instruction per cycle are called **superscalar** processors. Most modern processors support superscalar operation. 

**Single-Instruction, Multiple-Data (SIMD) Parallelism** ^7a80f9

At the lowest level, many modern processors have special hardware that allows a single instruction to cause multiple operations to be performed in parallel. This is called **SIMD** parallelism. 

These SIMD instructions are provided mostly to speed up applications that process image, sounds and video data. Although some compilers attempt to automatically extract SIMD parallelism from C programs, a more reliable method is to write programs using special **vector** data types supported in compilers such as GCC.

### The Importance of Abstractions in Computer Systems
The use of **abstractions** is one of the most important concepts in computer science. For example, one aspect of good programming practice is to formulate a simple API for a set of functions that allow programmers to use code without having to delve into its inner workings. Different languages provide different levels of abstraction.

We have already been introduced to several abstractions seen in computer systems, as indicated in Figure 1.18.

![](_attachments/Screenshot%202022-05-18%20at%2016.13.00.png)

One the processor side, the **instruction set architecture** provides an abstraction of the actual processor hardware. With this abstraction, a machine-code program behaves as if it were executed on a processor that performs just one instruction at a time. The underlying hardware is far more elaborate, executing multiple instructions in parallel - but always in a way that is consistent with the simple, sequential model. 

On the OS side, we have introduced three abstractions: *files* as an abstraction for I/O devices, *virtual memory* as an abstraction of program memory, and *processes* as an abstraction of a running program. To these abstractions, we add a new one: the **virtual machine**, providing an abstraction of the entire computer, the processor, and the programs. 
This idea has become prominent as a way to manage computers that must be able to run programs designed for multiple OSs (such as Windows, macOS, and Linux).