```toc
```

## The CPU
Most processors have the following special purpose, dedicated, registers:

**Program Counter (PC)**
> This register contains the address of the next instruction to be executed. The contents of the PC are automatically incremented each time an instruction is fetched.

**Stack Pointer (SP)**
> Processors need access to large amounts of external read/write RAM which facilitates temporary storage of data. The stack is a way of easily saving and restoring values in external memory. Processors usually have special instructions which allow you to push/pop values onto/from the stack.

**Processor Status (PS)**
> Instructions may yield results. For instance: "is the content of register X greater than the content of register Y?". The PS register holds this and other information about the state of the processor.

## Memory
Cache and main memory must be kept in step (coherent). In other words, if a word of main memory is held in one or more locations in cache, then the system must make sure the contents of the cache and main memory are the same. 

According to [Wikipedia](https://en.wikipedia.org/wiki/Memory_coherence), memory coherence is only a problem in multiprocessor systems. It's possible that two or more processors may simultaneously access the same memory location. Some scheme is required to notify all the processing elements of changes to shared values; such a scheme is called a **memory coherence protocol**.

## Buses
The system bus is divided into three logical functions:

* The Address Bus
* The Data Bus
* The Control Bus

The address bus specifies the addresses for data transfers.
The data bus holds the data transferred. It is bidirectional; it allows data to be read to/from the CPU.
The control bus contains various lines used to route timing and control signals throughout the system.

## Controllers and Peripherals
Peripherals are real devices, such as graphics cards or disks. They're controlled by controller chips on the system board or on cards plugged into it. These **controllers** are connected to the CPU and to each other by a variety of buses. The controllers are processors like the CPU itself; they can be viewed as intelligent helpers to the CPU.

All controllers are different but they usually have registers which control them. Software running on the CPU must be able to read/write those controlling registers.

To elaborate on this a bit further: CSAPP describes a distinction between **controllers** and **adapters**. The difference is mainly one of packaging. The key point is the their purpose is to transfer information between the I/O bus and an I/O device.

> [!Definition]
> **Controllers** are processors (like the CPU itself), whose purpose is to transfer information between an I/O device and the I/O bus. In other words, they *control* peripherals.

## Address Spaces
The **system bus** connects the CPU with main memory and is separate from the buses connecting the CPU to the system's hardware peripherals. Collectively, the memory space that the hardware peripherals exist in is called the **I/O space**.

The CPU can access both the system space memory and the I/O space memory. Whereas the controllers can only access system memory indirectly, and then only with the help of the CPU.

> [!INFO]
> *System memory* refers to main memory.

Typically, a CPU will have separate instructions for accessing the memory and I/O space.

There are times when controllers need to read or write large amounts of data directly to or from system memory. An example might be when user data is being written to a hard disk. In this case, **Direct Memory Access (DMA)** controllers are used to allow hardware peripherals to access system memory. But this access is under strict control and supervision of the CPU. ^7bdae2

## Timers
All operating systems need to know the time, and so PCs include a special peripherical called the **Real Time Clock (RTC)**. This provides two things: a reliable time of day and an accurate timing interval.
The RTC has its own battery so that it continues even when the system is turned off. This is how a PC always knows the correct data and time.