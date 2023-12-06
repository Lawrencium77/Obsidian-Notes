Procedures come in many forms:

* Functions
* Methods
* Subroutines
* Handlers
* [Callbacks](Event%20Loops.md#^b5c521)
* Etc.

There are many attributes that must be handled when providing machine-level support for procedures. Let's suppose a procedure `P` calls procedure `Q`, and `Q` then executes and returns back to `P`. These actions involve:

* **Passing Control**: The PC must be set to the starting address of `Q` upon entry, and then set to the instruction in `P` following the call to `Q` upon return.
* **Passing Data**: `P` must be able to provide parameters to `Q`, and `Q` must be able to return a value back to `P`.
* **Allocating and Deallocating Memory**: `Q` may need to allocate space for local variables, and free that storage before it returns.

The x86-64 implementation of procedures involves a combination of special instructions and a set of conventions on how to use machine resources. We will describe these step-by-step.

## The Run-Time Stack
While `Q` is executing, `P`, along with any of the procedures in the chain of calls up to `P`, is temporarily suspended. While `Q` is running, only it will need the ability to allocate new storage for its local variables, or to set up a call to another procedure. On the other hand, when `Q` returns, any local storage it has allocated can be freed. 

Therefore, a program can manage the storage required by its procedures using a **stack**.

The x86-64 stack grows towards the lower addresses, and the stack pointer `%rsp` points to the top element of the stack. Data can be stored on and retrieved from the stack using the `pushd` and `popq` instructions. 
When an x86-64 procedure requires storage beyond what it can hold in registers, it allocates space on the stack. This region is referred to as the procedure's **stack frame**. Figure 3.25 shows the overall structure of the run-time stack, including its partitioning into stack frames: ^a660fa

![](_attachments/Screenshot%202023-10-08%20at%2011.18.38.png)

When `P` calls `Q`, it pushes the **return address** onto the stack, indicating where within `P` the program should resume execution once `Q` returns. We consider the return address to be part of `P`'s stack frame. The code for `Q` allocates the space required for its stack frame by extending the current stack boundary. Within that space, it can do several things. First, it can save the current state of certain registers that `Q` is expecting to modify. When `Q` completes execution and returns control to `P`, these values are popped from the stack back into their respective registers.
Second, it can allocate space for local variables. Third, it can set up arguments for the procedures it calls. Procedure `P` can pass up to six integral values (i.e., pointers and integers) via the [general-purpose registers](Accessing%20Information.md); if `Q` requires more arguments, these can be stored by `P` in its stack frame prior to the call.

Many procedures have six or fewer arguments, meaning all of their parameters can be passed using registers. Thus, parts of the stack frame diagrammed above can be omitted. In fact, many functions don't even require a stack frame. This occurs when all of the local variables can be held in registers, and the function doesn't call any other functions. This is sometimes called a **leaf procedure**.

Return values from `Q` to `P` are passed via the registers, provided this value is small enough. For larger data structures, things get a little more complex. We study this later on.

## Control Transfers
Passing control from `P` to `Q` involves simply setting the PC to the starting address of the code for `Q`. However, when it later comes to time for `Q` to return, the processor must have some record of the code location where it should resume the execution of `P`. This information is recorded in x86-64 machines by invoking procedure `Q` with the instruction `call Q`. This pushes address `A` onto the stack and sets the PC to the beginning of `Q`. The pushed address `A` is the **return address** (as described above). The counterpart instruction `ret` pops address `A` off the stack and sets the PC to this value.

Figure 3.26 illustrates the execution of the `call` and `ret` instructions:

![](_attachments/Screenshot%202023-10-08%20at%2011.53.17.png)

The following are corresponding code excerpts:

![](_attachments/Screenshot%202023-10-08%20at%2011.54.30.png)

## Data Transfer
With x86-64, most data passing to and from procedures takes place via registers. When procedure `P` calls procedure `Q`, the code for `P` myst first copy the arguments into the proper registers. Similarly, when `Q` returns back to `P`, the code for `P` can access the returned value in register `%rax`. 

As a reminder: in x86-64, up to six integral arguments can be passed via registers. The registers are used in a specific order, with the name used for a register depending on the size and the data type being passed. These are shown in Figure 3.28:

![](_attachments/Screenshot%202023-10-08%20at%2012.04.18.png)

Note that this corresponds exactly to the [general-purpose register diagram](Accessing%20Information.md) seen previously. 

When a function has more than six integral arguments, the others are passed on the stack. Assume that `P` calls `Q` with $n > 6$ arguments. Then the code for `P` must allocate a stack frame with enough storage for arguments 7 through $n$, as illustrated in Figure 3.25. It copies arguments 1-6 into the appropriate registers, with argument 7 at the top of the stack. The program can then execute a call instruction to transfer control to `Q`. 

## Local Storage on the Stack
It's often the case that local data must be stored in memory instead of just the processor registers. Common reasons include:

* There aren't enough registers to hold all of the local data.
* The address-of operator `&` is applied to a local variable, and hence we must be able to generate an address for it.
* Some of the local variables are arrays/structs, and hence must be accessed by array/struct references. 

The portion of the stack frame for storing local variables is indicated in Figure 3.25.

## Local Storage in Registers
The set of program registers acts as a single resource shared by all procedures. Although only one procedure can be active at a given time, we must make sure when one procedure calls another, the **callee** doesn't overwrite some register value that the caller planned to use later. For this reason, x86-64 adopts a uniform set of conventions for register usage.

Registers `%rbx`, `%rbp`, and `%r12-%r15` are classified as **callee-saved** registers. When `P` calls `Q`, `Q` must preserve the values of these registers, ensuring they have the same values when `Q` returns to `P` as they did when `Q` was called. Procedure `Q` can preserve this value by either not changing it at all, or by pushing the original value on the stack, altering it, and then popping the old value from the stack before returning. This is the purpose of the **Saved Registers** section in Figure 3.25.

All other registers, except for the stack pointer, are classified as **caller-saved** registers. This means they can be modified in any function. In the context of `P` calling `Q`, this means it is incumbent upon `P` (the caller) to first save local data stored in these registers before it calls `Q`.

## Recursive Procedures
The conventions we have described above allow x86-64 procedures to call themselves recursively. Figure 3.35 shows both the C code and generated assembly code for a recursive factorial function:

![](_attachments/Screenshot%202023-10-08%20at%2012.39.49.png)

We can see that the assembly code uses register `%rbx` to hold the parameter `n`, after first saving the existing value on the stack (line 2). It later restores the value before returning (line 11). 

The key idea here is that this example shows that calling a function recursively proceeds just like any other function call. 









