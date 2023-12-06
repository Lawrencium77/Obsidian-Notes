An x86-64 CPU contains a set of 16 **general-purpose registers**, storing 64-bit values. Figure 3.2 shows a diagram of these registers:

![](_attachments/Screenshot%202023-09-29%20at%2016.47.06.png)

> [!INFO]
> Note that these are only the general-purpose registers. There are others, such as the instruction pointer, control registers, etc.

Their names all begin with `%r`, but otherwise follow different naming conventions due to historical reasons. For instance, the original 8086 had eight 16-bit registers, shown in Figure 3.2 as `%ax` through `%bp`. 

As the nested boxes in Figure 3.2 indicate, instructions can operate on data of different sizes stored in the low-order bytes of the 16 registers. Later on, we will show instructions for copying and generating 1, 2, 4, and 8-byte values. When these instructions have registers as destinations, two conventions arise for what happens to the remaining bytes:

> 1. Those that generate 1 or 2-byte quantities leave the remaining bytes unchanged.
> 2. Those that generate 4-byte quantities set the upper 4 bytes to zero.

The annotations along the right-hand side reflect that **different registers serve different roles in typical programs**. Most unique is the **stack pointer**, `%rsp`, used to indicate the end position in the run-time stack. The other 15 registers have more flexibility in their uses. 

### Operand Specifiers
Most instructions have **operands** specifying the source values to use, and destination locations, in which to place results. Source values can be given as constants, read from registers, or read from memory. Results can be stored in registers or memory.

Thus, the different operands can be classified into three types: **immediate**, **register**, and **memory**. These are shown below:

![](_attachments/Screenshot%202023-09-29%20at%2016.57.31.png)

The second column indicates the notation used to indicate some operand, and the third column shows how we indicate its value.

There are many different **addressing modes** allowing different forms of memory references. The most general form is shown at the bottom, with syntax $Imm(r_b, r_i, s)$. The other forms are simply special cases of this form. This isn't complicated; the idea is that we map some values $r_b, r_i, s$ to a memory address $Imm + R(r_b) + \dots$ and use this address to obtain some value.

## Data Movement Instructions
We present a number of different data movement instructions. We group these into **instruction classes**, where the instructions in a class perform the same operation but with different operand sizes.

Figure 3.4 lists the simplest form of data movement instructions - the `MOV` class. These simply move data from one place to another:

![](_attachments/Screenshot%202023-09-29%20at%2017.07.44.png)

x86-64 imposes the restriction that a move instruction cannot have both operands refer to memory locations. 

Referring to figure 3.2, register operands can be the labeled portions of any of the 16 registers, where the size of the register must match the size designated by the last character of the instruction. 

The following examples show the five possible combinations of source and destination types:

![](_attachments/Screenshot%202023-09-29%20at%2017.12.12.png)

Figures 3.5 and 3.6 document two classes of data movement instructions for use when copying a smaller source value into a larger destination:

![](_attachments/Screenshot%202023-09-29%20at%2017.13.51.png)

![](_attachments/Screenshot%202023-09-29%20at%2017.14.06.png)

Instructions in the `MOVZ` class fill out the remaining bytes of the destination with zeros; those in `MOVS` fill them out by sign extension, replicating copies of the most significant bit of the source operand. 

## Data Movement Example
As an example of code that uses data movement instructions, consider the data exchange routine shown in Figure 3.7:

![](_attachments/Screenshot%202023-09-29%20at%2017.20.10.png)

When the procedure beings execution, parameters `xp` and `y` are stored in registers `%rdi` and `%rsi`. This is standard (these registers are always used for this purpose). Instruction 2 then reads `x` from memory and stored the value in `%rax`. Again, `%rax` is always used for this purpose - to store the return value. 

Two features are worth noting. First, pointers are just addresses. Dereferencing a pointer involves copying that pointer in a register, and then using this register in a memory reference. Second, local variables such as `x` are often kept in registers rather than stored in memory locations.

## Pushing and Popping Stack Data
The final two data movement operations are used to push data onto and pop data from the program stack. The stack plays a vital role in handling procedure calls. The stack is stored in some region of memory:

![](_attachments/Screenshot%202023-09-29%20at%2017.49.25.png)

The stack grows downwards such that the top element of the stack has the lowest address of all stack elements. The stack pointer `%rsp` holds the address of the top stack element. 

The push and pop instructions are as follows:

![](_attachments/Screenshot%202023-09-29%20at%2017.50.36.png)

Both instructions act of 64-bit data, and take a single operand - the data source/destination. 

Pushing a quad word onto the stack involves decrementing the stack pointer by 8 and then writing the value at the new top-of-stack address. Therefore, `pushq %rbp` is equivalent to:

![](_attachments/Screenshot%202023-09-29%20at%2017.52.22.png)

except that the `pushq` instruction is encoded in machine code as a single byte, whereas the pair of instructions above requires a total of 8 bytes. 

Since the stack is contained in the same memory space as other forms of program data, programs can access arbitrary positions within the stack using the standard memory addressing methods.
