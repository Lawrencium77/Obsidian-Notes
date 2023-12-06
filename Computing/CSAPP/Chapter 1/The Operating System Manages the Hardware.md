Back to our `hello` example. When the shell loaded and ran the `hello` program, and when the `hello` program print its message, neither program accessed the keyboard, display, disk or main memory directly. Rather, they relied on the services provided by the **operating system (OS)**. We can think of the OS as a layer of software interposed between the  application program and the hardware, as shown in Figure 1.10. All attempts by an application program to to manipulate the hardware must go through the OS.

![](_attachments/Screenshot%202022-05-13%20at%2012.57.24.png)

The OS has two primary purposes: (1) to protect the hardware from misuse by runaway applications and (2) to provide applications with simple and uniform mechanisms for manipulating complicated and often wildly different low-level hardware devices. 

The OS achieves both goals via the fundamental abstractions shown in Figure 1.11: **processes, virtual memory** and **files**. As this figure suggests, files are abstractions for I/O devices, virtual memory is an abstraction for both the virtual memory and main memory and processes are an abstraction for the processor, main memory and I/O devices. We will discuss each in turn.

![](_attachments/Screenshot%202022-05-13%20at%2013.01.05.png)

### Processes

When a program like  `hello`  runs on a modern system, the OS provides the illusion that the program is the only one running on the system. The program appears to have exclusive use of the processor, main memory, and I/O devices. The processor appears to execute the instructions in the program, on after the other, without interruption. And the code and data of the program appear to be the only objects in the system's memory. These illusions are provided by the notion of a process - one of the most important and successful ideas in computer science.

A *process* is the OS's abstraction for a running program. Multiple processes can run concurrently on the same system, and each process appears to have exclusive use of the hardware. By **concurrently**, we mean that the instructions of one process are interleaved with the instructions of another process. In most systems, there are more processes to run than there are CPUs to run them.

Traditional systems could only execute one program at a time, while newer *multi-core* processors can execute several programs simultaneously. In either case, a single CPU can appear to execute multiple processes concurrently by having the processor switch among them. The OS performs this interleaving with a mechanism called **context switching**. To simplify the rest of the discussion, we consider only a *uniprocessor* system containing a single CPU. We will discuss *multiprocessor* systems later in this chapter.

The OS keeps track of all the state information that the process needs in order to run. This state, which is known as the **context**, includes information such as the current value of the PC, the register file, and the contents of main memory. At any point in time, a uniprocessor system can only execute the code for a single process. When the OS decides to transfer control from the current process to some new process, it performs a *context switch* by saving the context of the current process, restoring the context of the new process, and then passing control to the new process. The new process picks up exactly where it left off. This idea is shown in Figure 1.12:

![](_attachments/Screenshot%202022-05-17%20at%2008.36.38.png)

There are two concurrent processes in our example scenario: the shell process and the  `hello`  process. Initially, the shell process is running alone, waiting for input on the command line. When we ask it to run  `hello` , the shell carries out our request by invoking a special function known as a **system call** that passes control to the OS.  The OS saves the shell's context, creates a new  `hello`  process and its context, and then passes control to the new  `hello`  process. After  `hello`  terminates, the OS restores the context of the shell process and passes control back to it, where it waits for the new command-line input.

As Figure 1.12 indicates, the transition from one process to another is managed by the OS **kernel**. The kernel is the portion of the OS code that is always resident in memory. When an application program requires some action by the OS, such as to read of write a file, it executes a special **system call** instruction, transferring control back to the kernel. The kernel then performs the requested operation and returns back to the application program. Note that the kernel is not a separate process. Instead, it is a collection of code and data structures that the system uses to manage all the processes.

Implementing the process abstraction requires cooperation between the low-level hardware and the OS software. This is explored more in Chapter 8.

### Threads
Although we normally think of a process as having a single control flow, in modern systems a process can actually consist of multiple execution units, called **threads**. Each runs in the context of the process and shares the same code and global data. Threads are an increasingly important programming model because of the requirement for concurrency in network servers, because it is easier to share data between multiple threads than between multiple processes, and because threads are typically more efficient than processes. Multi-threading is also one way to make programs run faster when multiple processors are available. The basics of concurrency, including how to write threaded programs, are discussed in Chapter 12.

### Virtual Memory

^90ee75

*Virtual memory* is an abstraction that provides each process with the illusion that it has exclusive use of the main memory. Each process has the same uniform view of memory, which is known as its **virtual address space**. The virtual address space for Linux process is shown in Figure 1.13:

![](_attachments/Screenshot%202022-05-17%20at%2008.50.30.png)

The topmost region is reserved for code and data in the OS that is common to all processes. The lower region holds the code and data defined by the user's process. Note that the addresses in the figure increase from bottom to top.

The virtual address space seen by each process consists of a number of well-defined areas. You'll learn more about these areas later in the book, but it will be helpful to briefly look at each:

* **Program code and data** 
	* Code begins at the same fixed address for all processes, followed by data locations that correspond to global C variables. The code and data areas are initialised directly from the contents of an executable object file. 
* **Heap** 
	* Next is the run-time **heap**. Unlike the code and data areas, which are fixed in size once the process begins running, the heap expands and contracts dynamically at run time as a result of calls to C standard library routines such as `malloc` and `free`. 
* **Shared libraries** 
	* Near the middle of the address space is an area that holds the code and data for shared libraries such as the C standard library. The notion of a shared library is a powerful but somewhat difficult concept. 
* **Stack** 
	* At the top of the user's virtual address space is the **user stack** that the compiler uses to implement function calls. Like the heap, the user stack contracts and expands dynamically during the execution of the program. Each time we call a function, it grows; each time we return from a function, it contracts.
* **Kernel virtual memory** 
	* The top region of the address space is reserved for the kernel. Application programs are not allowed to read or write the contents of this area or to directly call functions defined in kernel code. Instead, they must invoke the kernel to perform these operations.

For virtual memory to work, a sophisticated interaction is required between the hardware and OS software, including a hardware translation of every address generated by the processor. The basic idea is to store the contents of a processes' virtual memory on disk and then use main memory as a cache for the disk. 

### Files
A **file** is a sequence of bytes - nothing more and nothing less. Every I/O device, including disks, keyboards, displays, and even networks, is modelled as a file. All input and output in the system is performed by reading and writing files, using as small set of system calls known as **Unix I/O**.

This notion of a file is powerful because it provides applications with a uniform view of all the varied I/O devices that might be contained in the system. For example, application programmers who manipulate the contents of a disk file are blissfully unaware of the specific disk technology. Further, the same program will run on different systems that use different disk technologies.
