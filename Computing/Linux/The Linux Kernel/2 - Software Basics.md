```toc
```

## Computer Languages

### Assembly Languages
Assembly languages explicitly handle registers and operations on data. They are specific to a particular microprocessor. 

They are tedious and prone to errors! Very little of the Linux kernel is written in assembly language. Those parts that are are written only for efficiency, and are specific to particular microprocessors.

## The C Programming Language and Compiler
I skipped this part.

## What is an Operating System?
An operating system abstracts the real hardware of the system and presents the system's users and its applications with a virtual machine. Linux is made up of a number of functionally separate pieces that, together, comprise the OS. One obvious part of Linux is the kernel itself, but even that would be useful without libraries or shells.

In order to start understanding what an OS is, consider what happens when you type an apparently simple command: 

![](_attachments/Screenshot%202022-12-05%20at%2020.47.23.png)

The `$` is a prompt put out by a login shell (in this case, `bash`). This means that it is waiting for the user to type some command. Typing `ls` causes the keyboard **driver** to recognise that characters have been typed. The keyboard driver passes them to the shell which processes that command by looking for an executable image of the same name.

> [!INFO]
> Loosely speaking, a **driver** is a piece of software that controls some device.

It finds that imagine in `/bin/ls`. Kernel services are called to pull the `ls` executable image into virtual memory and start executing it. The `ls` image makes calls to the file subsystem of the kernel to find out what files are available. The filesystem might make use of cached filesystem information or use the disk device driver to read this information from disk. It might even cause a network driver to exchange information with a remote machine to found out details of remote files that this system has access to (filesystems can be remotely mounted via the **Networked File System (NFS)**).
Whichever way the information is located, `ls` writes that information out and the video driver displayers it on screen.

All of this seems a complicated. But it shows that even the most simple commands reveal that an OS is in fact a co-operating set of functions that together give the user a coherent view of the system.

### Memory Management
One of the basic tricks of an OS is the ability to make a small amount of physical memory *behave* like it's bigger than it is. This apparently large memory is called **virtual memory**. 

The system divides the memory into easily handled **pages** and **swaps** these pages onto a hard disk as the system runs. The software doesn't notice because of another trick: **multi-processing**.

> [!INFO]
> A memory **page** is a fixed-length contiguous block of virtual memory. It's the smallest unit of data for memory management in a virtual memory operating system.
> A transfer of pages between main memory and an auxiliary store (such as a hard disk) is referred to as **paging** or **swapping**.

^40d765

### Processes
**A process could be thought of as a program in action**. Each process is a separate entity that is running a particular program.

Operating Systems run multiple processes simultaneously by running each process for a short period. This period of time is called a **time-slice**. This trick is known as **multi-processing**. Processes are protected from one another so that if one process crashes or malfunctions then it doesn't affect any others. The OS achieves this by giving each process a separate address space.

### Device Drivers
Device drivers make up a major part of the Linux kernel. They control the interaction between the OS and the hardware device they are controlling.
For example, the filesystem makes use of a general block device interface when writing blocks to disk. The driver takes care of the details and makes device-specific things happen.

Device drivers are specific to the controller chip that they are driving.

### Filesystems

In a loose sense, a **filesystem** is the structure and logic rules used to manage groups of files. Just like in the physical setting.

According to Wikipedia:

> [!QUOTE]
> A filesystem is a method and data structure that the operating system uses to control how data is stored and retrieved. Without a file system, data placed in a storage medium would be one large body of data with no way to tell where one piece of data stopped and the next began.
> 
> There are many kinds of file system, each with unique structure and logic, properties of speed, flexibility, security, and more.


In Linux, the separate filesystems that the system may use are not accessed by device identifiers but instead are combined into a single hierarchical tree structure that represents the filesystem as a single entity.

Linux adds each new filesystem into this single filesystem tree as they are **mounted** onto a **mount** directory, for example `/mnt/cdrom`. One of the most important features of Linux is its support for many different filesystems.

> [!INFO]
> I thought I'd add a bit more info about *mounting*.
> According to Wikipedia:
> 
> "Unix-like systems assign a device name to each device, but this is not how the files on that device are accessed. Instead, to gain access to files on another device, the operating system must first be informed where in the directory tree those files should appear. This process is called **mounting** a file system. For example, to access the files on a CD-ROM, one must tell the operating system "*Take the file system from this CD-ROM and make it appear under such-and-such directory.*" The directory given to the operating system is called the _[mount point](https://en.wikipedia.org/wiki/Mount_point "Mount point")_ – it might, for example, be `/media`."

Linux presents all of the mounted filesystems as one integrated virtual filesystem. This means that users and processes don't need to know what sort of filesystem any file is part of; they can just use them.

The block device drivers hide the differences between the physical block device types and, so far as each filesystem is concerned, the physical devices are just linear collections of blocks of data. The block sizes may vary between devices (e.g. 512 bytes used to be common for floppy disks, and 1024 bytes for hard disks) and again, this is hidden from the users of the system.

> [!INFO]
> *Block* device drivers are different from *character* device drivers. I am not too fussed about their difference at this stage.

## Kernel Data Structures
Data structures contain data and pointers; addresses of other data structures or the addresses of routines. 

### Linked Lists
Linux uses a number of **linked** data structures. In a linked list, each element is a data structure. There is also an associated pointer that contains the address of the next data structure.

### Hash Tables
A hash table can be thought of as an array of pointers. Each pointer gives the address to a data structure. Of course, the index of the pointer in the array is derived from the value in the corresponding data structure.

Linux often uses hash tables to implement caches. 

### Abstract Interfaces
I skipped this part, as I didn't understand it properly.
