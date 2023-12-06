At this point, our `hello.c` program has been translated by the compilation system into an executable called `hello` that is stored on disk. To run the executable on a Unix system, we type its name to an application program known as **shell**:

```linux
linux> ./hello
hello, world
linux>
```
The shell is a command-line interpreter that prints a prompt, waits for you to type a command line, and then performs the command. If the first word of the command line doesn't correspond to a built-in shell command, then the shell assumes this is the name of an executable that it should load and run. So in this case, the shell loads and runs the `hello`  program and then waits for it to terminate. Once the program has finished, the shell prints a prompt and waits for the next input command line.

### Hardware Organisation of a System
To understand what happens to our `hello`  program when we run it, we need to understand the hardware organisation of a typical system, which is shown in Figure 1.4.

![](_attachments/Screenshot%202022-05-13%20at%2011.46.08.png)

We need not worry about the detail of this diagram just now. We'll get to its various details later in the book.

**Buses**
Running through the system is a collection of electrical conduits called **buses**. They carry information between components. Buses are typically designed to transfer fixed-size chunks of bytes called **words**. The number of bytes in a word (**word size**) is a fundamental system parameter that varies across systems. Most machines have word sizes of either 4 or 8 bytes.

**I/O Devices**
Input/output (I/O) devices are the system's connection to the outside world. Our example system has four I/O devices: a keyboard, mouse, display, and disk. Initially, the `hello` program resides on the disk.
Each I/O device is connected to the I/O bus by either a **controller** or **adapter**. The distinction between the two is mainly one of packaging: controllers are chip sets in the device itself or on the main printed circuit board (**motherboard**). An adapter is a card that plugs into a slot on the motherboard. Regardless, the purpose of each device is to transfer information between the I/O bus and an I/O device.

**Main Memory**
The **main memory** is a temporary storage device that holds both a program and the data it manipulates while the processor is executing the program. Physically, it consists of a collection of DRAM chips. Logically, memory is organized as a linear array of bytes, each with its own unique address (array index) starting at zero.

**Processor**
The **central processing unit (CPU)**, or simply **processor**, is the engine the **executes** instructions stored in main memory. At its core is a word-size storage device (or **register**) called the **program counter (PC)**. At any point in time, the PC points at some machine-language instruction in main memory. 
From the time the power is applied to the system until its shut off, a processor repeatedly executes the instruction pointed at by the PC and updates the PC to point to the next instruction (which may or may not be contiguous in memory to the instruction that was just executed).

Put simply, the PC indicates where a processor is in its program sequence.

There are only a few simple operations that a processor can perform. These revolve around the main memory, the **register file**, and the **arithmetic/logical unit (ALU)**. The register file is a small storage device that consists of a collection of word-size registers, each with its own unique name. The ALU computes new data and address values. Here are examples of the operations that the CPU might carry out:

* **Load:** Copy a byte of a word from main memory into a register.
* **Store:** Copy a byte or word from a register to a location in main memory.
* **Operate:** Copy the contents of two registers to the ALU, perform and arithmetic operation on the two words, and store the results in a register.
* **Jump:** Puts a new address into the PC.

A processor *appears* to operate according to a very simple instruction execution model, defined by its **instruction set architecture**. However, modern processors use far more complex mechanisms to speed up program execution. Thus, we can distinguish the processor's instruction set architecture (describing the set of machine-code instructions) from its **microarchitecture**, describing how the processor is actually implemented. In this sense, the machine's instruction set architecture provides a useful abstraction.

### Running the `hello` Program

Given this simple view of a system's hardware organisation and operation, we can begin to understand what happens when we run our program. We must omit a lot of details here that will be filled in later, but for now we will be content with the big picture.

Initially, the shell program is executing its instructions, waiting for us to type a command. As we type the characters `./hello` at the keyboard, the shell program reads each one into a register and then stores it in memory, as shown in Figure 1.5.

![](_attachments/Screenshot%202022-05-13%20at%2012.10.01.png)
When we hit `enter`, the shell knows that we have finished typing the command. The shell then loads the executable `hello` file by executing a sequence of instructions that copies the code and data in the `hello` object file from disk to main memory. 

Using a technique known as **direct memory access (DMA)**, the data travel directly from disk to main memory, without passing through the processor. This step is shown in Figure 1.6. ^515cc8

![](_attachments/Screenshot%202022-05-13%20at%2012.13.12.png)

Once the code and data in the `hello` object file are loaded into memory, the processor begins executing the machine-language instructions in the `hello` program's `main` routine. These instructions copy the bytes in the `hello, world\n` string from memory to the register file, and from there to the display device. This step is shown in Figure 1.7.

![](_attachments/Screenshot%202022-05-13%20at%2012.14.56.png)
