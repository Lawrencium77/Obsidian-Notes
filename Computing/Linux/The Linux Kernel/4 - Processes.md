This chapter describes what a process is and how the Linux kernel creates, manages and deletes the processes in a system.

A program is a set of machine code instructions and data stored in an executable image on disk. As such, it is a passive entity. A process can be thought of as a program in action.

It is dynamic, constantly changing as the machine code instructions are executed by the processor. As well as the program's instructions and data, the process also includes the program counter and all of the CPU's registers as well as the process stacks containing temporary data.

Processes are separate tasks, each with their own rights and responsibilities. Each individual process runs in its own virtual address space and is not capable of interacting with another process except through secure, kernel managed mechanisms.

During the lifetime of a process, it will use many system resources. It will use the CPUs to run its instructions and the system's physical memory to hold it and its data. It will open and use files within the filesystems and may use the physical devices in the system. Linux must keep track of the process such that it can manage it and the other processes in the system.

Linux is a multiprocessing OS. Its objective is to have a process running on each CPU at all times. If there are more processes than CPUs (there usually are), then other processes must wait until a CPU becomes free.
Multiprocessing is a simple idea: a process is executed until it must wait, usually for some system resource. To maximise CPU utilisation, many processes are kept in memory simultaneously. Whenever a process needs to wait the OS takes the CPU away from that process and gives it to another process that can execute immediately. 
It is the **scheduler** which chooses the most appropriate process to run next.

## Linux Processes
In Linux, each process is represented by a `task_struct` data structure (task and process are terms that are used interchangeably). There also exists a `task` vector, which is an array of pointers to every `task_struct` data structure in the system.

Although the `task_struct` data structure is quite large and complex, its fields can be divided into a number of functional areas:

#### State
As a process executes, it changes *state* according to circumstances. Linux processes have the following states:

1. **Running (R)** : The process is either running, or is ready to run (it's waiting to be assigned to a system CPU).
2. **Waiting (D/S)**: The process is waiting for an event or resource. Linux differentiates between two types of waiting processes: **interruptible** and **uninterruptible**. Interruptible waiting processes can be interrupted by signals, whereas uninterruptible waiting processes are waiting directly on hardware conditions and cannot be interrupted.
3. **Stopped (T)**: The process has been stopped, usually by receiving a signal. 
4. **Zombie (Z)**: This is a halted process that, for some reason, still has a `task_struct` data structure in the `task` vector. It's a dead process. These often occur when the parent process has failed to remove it from `task`. ^272c93

A bit more on zombie processes: When a process has completed its execution or is terminated, it sends the SIGCHLD signal to its parent process and goes into zombie state. The zombie process remains in this state until the parent process clears it from the process table. 

#### Scheduling Information
The scheduler needs this information in order to decide which process in the system should be run next.

#### Identifiers
Every process in the system has a process identifier. It is **not** an index into the `task` vector, but is simply a number.

#### Inter-Process Communication
Linux supports the classic Unix mechanisms of signals, pipes, and semaphores. The IPC mechanisms supported by Linux are described in the [relevant chapter](5%20-%20Interprocess%20Communication%20Mechanisms.md).

#### Links
In Linux, no process is independent of any other process. Each process in the system (except the initial process) has a parent process. New processes are not created - they are **cloned** from previous processes. Every `task_struct` representing a process keeps pointers to its parent process and to its siblings, as well as to its own child processes.

#### Times and Timers
The kernel keeps track of a process creation time as well as the CPU time that it consumes during its lifetime.

#### File System
Processes can open and close files as they wish. The process `task_struct` contains pointers to descriptors for each open file. See [below](#^d3a18b) for more on file descriptors.

#### Virtual Memory
Not much detail here.

#### Process Context
A process can be thought of as the sum total of the system's current state. When a process is suspended, all of the CPU-specific context must be saved in the `task_struct` for the process. 


## Identifiers
Linux uses user and group identifiers to check for access rights to file and images in the system. All of the files in a Linux system have ownerships and permissions.

Groups are Linux's way of assigning privileges to files and directories for a group of users, rather than to a single user of to all processes in the system. A process can belong to several groups and these are held in the `groups` vector in the `task_struct` for each process. So long as a file has access rights for one of the groups that a process belongs to then that process will have appropriate group access rights to that file.


## Scheduling
All processes run partially in **user mode** and partially in **system mode**. User mode has far fewer privileges than system mode. Each time a process makes a **system call**, it swaps from user mode to system mode and continues executing. At this point, the kernel is executing on behalf of the process. 

> [!REMINDER]
> A **system call** is the mechanism by which a computer program requests a service from the kernel of an operating system.

In Linux, processes do not preempt the current, running process. Each process decides to relinquish the CPU that it is running on when it has to wait for some system event.

Processes are always making system calls and so often need to wait. Even so, if a process executes until it waits then it might use a disproportionate amount of CPU time. Linux therefore uses pre-emptive scheduling. In this scheme, each process is allowed to run for a small amount of time and when this time has expired, another process is selected to run. This small amount of time is called a **time-slice**. ^4966f8

It is the **scheduler** that selects the most deserving process to run out of all runnable processes in the system.

Linux uses a reasonably simple priority-based scheduling algorithm to choose between the current processes in the system.

> I skipped a decent chunk more info on scheduling.

## Files

> [!INFO]
> CSAPP had some interesting extra info on how files are treated in Linux. I made a few [notes](../../CSAPP/Chapter%2010/Sharing%20Files.md) on the key bits.

There are two data structures that describe file system specific information for each process in the system (see figure below). The first is `fs_struct`.  It contains pointers to the process's VFS inodes.

The second, `files_struct`, contains information about all of the files that the process is currently using. Programs read from **standard input** and write to **standard output**. Any error messages should go to **standard error**. These may be files, terminal input/output or a real device. But so far as the program is concerned, they are all treated as files.

![](_attachments/Screenshot%202022-12-12%20at%2017.46.05.png)

> [!INFO]
> An [inode](https://en.wikipedia.org/wiki/Inode) is a data structure that describes a file. It holds the block locations for the file's data. It also contains metadata about the file, such as owner and permission data.

Every file has its own descriptor and the `files_struct` contains pointers to multiple `file` data structures, each one describing a file being used by this process. The `f_mode` field describes what mode the file has been created in. `f_pos` holds the position in the file where the next read/write will occur. `f_inode` points to the VFS inode describing the file and `f_ops` is a pointer to a vector of routine addresses; one for each function you might wish to perform on a file. There is, for example, a write data function.

Every time a file is opened, one of the free `file` pointers in the `files_struct` is used to point to the new `file` structure. Linux processes expect three **file descriptors** to be open when they start. These are stdout, stdin and stderr. These are usually inherited from the creating parent process. All accesses to files are via standard system calls which pass or return file descriptors.
These descriptors are indices in the process's `fd` vector, so stdin, stdout and stderr have file descriptors 0, 1 and 2. Each access to the file uses the `file` data structure's file operation routines, as well as the Virtual File System (VFS) inode, to achieve its needs.

> [!Key Point]
> File descriptors are indices into a process's `fd` vector, that's contained within its `files_struct`.


## Virtual Memory
A processes's virtual memory contains executable code and data from many sources:
- First, there is the program image that is loaded; for example, a command like `ls`. 
- Secondly, processes can allocate virtual memory to use during their processing, say to hold the contents of files that it's reading. This newly allocated virtual memory needs to be linked into the processes's existing virtual memory such that it can be used. 
- Thirdly, Linux processes use libraries of commonly useful code, for example file handling routines. It doesn't make sense for each process to have its own copy of the library, so Linux has **shared libraries** that can be used by several running processes at the same time. The code and data from these shared libraries must be linked into this processes's virtual address space (and into the virtual address space of other processes sharing the library).

> [!INFO]
> A program *image* is simply a machine code representation of a program that is ready to be executed. It's the result of the compilation process.

At any given time, a process will not have used all the code and data contained in its virtual memory. It could contain code that's only used during certain situations. It may only use some of the routines from its shared libraries. 
It would be wasteful to load all this code & data into physical memory. Multiply this wastage by the number of processes in the system and it would be very inefficient indeed. Instead, Linux uses a technique called **demand paging** where the virtual memory of a process is brought into physical memory only when a process attempts to use it. So, instead of loading the code and data into physical memory straight away, the Linux kernel alters the process's page table, marking the virtual areas as existing but not in memory. When a process attempts to access the code or data, the system hardware will generate a **page fault** and hand control to the Linux kernel to fix things up. This means that for every area of virtual memory in the process's address space, Linux needs to know where that virtual memory comes from and how to get it into memory so that it can fix these page faults. ^c69d45

> [!INFO]
> A **[page](2%20-%20Software%20Basics#^40d765) table** is a data structure used to store information about the pages in a memory. It contains the mapping between virtual addresses and physical addresses.

The contents of each process's virtual memory is described by a `mm_struct` data structure pointed to from it's `task_struct`. The `mm_struct` data structure also contains information about the loaded executable image and a pointer to the process's page tables. It contains a pointer to a linked list of `vm_area_struct` data structures, each representing an area of virtual memory within this process.

This linked list is in ascending virtual memory order. Figure 4.2 shows these ideas:

![](_attachments/Screenshot%202022-12-12%20at%2018.13.01.png)


## Creating a Process
When a system starts up, it is running in kernel mode. There is only one process: the initial process. At the end of system initialization, the initial process starts up a kernel thread (called `init`) and then sits in an idle loop doing nothing. Whenever there is nothing else to do the scheduler will run this idle process.

The `init` kernel thread or process has a PID of 1. It does does some initial setting up of the system (such as mounting the root file system) and then executes the system initialization program. This is one of `/etc/init`, `/bin/init` or `/sbin/init` depending on your system. 

New processes are created by cloning old processes, or rather by cloning the current process. A new task is created by a system call (**fork** or **clone**), and the cloning happens with the kernel in kernel mode. At the end of the syscall there is a new process waiting to run once the scheduler chooses it. The new `task_struct` is entered into the `task` vector and the contents of the old (`current`) process's `task_struct` are copied into the cloned `task_struct`.

When cloning processes, Linux allows the two processes to share resources. This applies to the processes's files, signal handlers and virtual memory. When the resources are to be shared, their respective `count` fields are incremented such that Linux will not deallocate these resources until both processes have finished using them.
For example, if the cloned process is to share virtual memory then it's `task_struct` will contain a pointer to the `mm_struct` of the original process and that `mm_struct` has its `count` field incremented to show the number of current processes sharing it.

Cloning a process's virtual memory is quite difficult. Linux uses a technique called **"copy on write"** which means that virtual memory will only be copied when one of the two processes tries to write to it. 


## Executing Programs
In Linux, programs and commands are normally executed by a command interpreter. A command interpreter is a user process like any other, and is called a **shell**.

> [!KEY POINT]
> A shell is a command-line interpreter.

^e21269

With the exception of a few built-in commands (like `cd` and `pwd`), a command is an executable binary file. For each command entered, the shell searches the directories in its `$PATH` environment variable for an executable image with a matching file. If a file is found, it's loaded and executed. The shell clones itself using the **fork** mechanism described above and then the new child process replaces the binary image that it was executing, the shell, with the contents of the executable image just found.
Normally the shell waits for the command to complete, or rather for the child process to exit. You can cause the shell to run again by pushing the child process to the background. This is done by typing `control-Z`, which causes a `SIGSTOP` to be sent to the child process, stopping it. You then use the shell command `bg` to push it into a background, the shell sends it a `SIGCONT` signal to restart it, and it remains in this state until it ends or needs terminal input/output.

> [!INFO]
> A shell **built-in** is a command that is executed directly in the shell itself, instead of an external executable program which the shell would load and execute.
> They're used for commands that are frequent or closely tied with the shell itself.
> They work significantly faster than external programs since there is no program loading overhead.

I took another paragraph from this section and turned it into some short notes on [shebangs](../Shebangs.md).
I also made some notes on what happens in the specific instance of running a [shell script](../Running%20Shell%20Scripts.md).













