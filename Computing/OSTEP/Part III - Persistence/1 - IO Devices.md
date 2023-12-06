> CRUX: How to Integrate I/O Into Systems

## System Architecture

CPUs use a hierarchical bus structure.

Because faster buses must necessarily be shorter. So high-performance memory buses don't have much room to plug devices into it.

## A Canonical Device

![](_attachments/Screenshot%202023-11-23%20at%2021.25.01.png)

The hardware device presents an interface to the rest of the system. It then has its own internal structure.

## The Canonical Protocol

A typical interaction that the OS undergoes with the device would be:

![](_attachments/Screenshot%202023-11-23%20at%2021.26.24.png)

When the main CPU is involved with data movement, we call it **programmed IO**.

The periods of waiting above are called **polling**. They're obvs not good.

## Using Interrupts Instead

Interrupts mean we can avoid polling. E.g:

![](_attachments/Screenshot%202023-11-23%20at%2021.28.34.png)

1 and 2 refer to different processes.

Interrupts aren't *always* the best solution. E.g., if a device does its task very quickly, then the context switch overhead and interrupt handling is more expensive than just polling. 

Another interrupt-based optimisation is **coalescing** - the device which needs to interrupt waits for a bit before sending to the CPU. While waiting, other requests may complete. So multiple interrupts can be coalesced into a single delivery, lowering the overhead of interrupt processing.

## DMA
[DMA](1%20-%20Hardware%20Basics#^7bdae2) means the CPU can avoid having to execute instructions to move data between memory and the device.

## Methods of Device Interaction
[Memory-mapped IO](../../CSAPP/Chapter%209/Memory%20Mapping.md) is only one way for the CPU to interact with a device.
It can also have explicit **I/O Instructions** to specify a way for the OS to send data to specific device registers.

## Device Drivers

> Crux: How to fit devices, each of which have very specific interfaces, into the OS? We'd like to keep the OS as general as possible.

We can view this as a form of **abstraction**. 
At the lowest level, a piece of software in the OS must know in detail how a device works. This is the [driver](../../Device%20Driver%20Types.md). 

The figure below shows the Linux file system software stack:

![](_attachments/Screenshot%202023-11-23%20at%2021.38.17.png)

The file system is oblivious to the specifics of which disk class it's using. It simply issues read and write requests. These are then routed to the appropriate driver, which handles the details.

Since drivers are required for any device used in a system, over time they represent a large fraction of OS kernel code:

> [!INFO]
> **Fun Fact**: Over 70% of Linux kernel code is found in device drivers.
> 









