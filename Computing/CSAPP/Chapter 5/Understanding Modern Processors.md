Over the previous few sub-chapters, we've achieved a considerable improvement in our [Program Example](Program%20Example.md). We would now like to see just what factors are constraining code performance and how we can improve further.

As we do so, we must consider optimisations that exploit the processor **microarchitecture**. Each processor is different but we can apply some basic optimisations that will yield results on a broad class of processors.

Modern processors are extremely complex. One consequence of this is that their actual operation is far different from the view that is perceived by looking at machine-level programs. [Instruction-Level Parallelism](Important%20Themes#^12c8c3) is an example of this.

Although the detailed design of microprocessors is beyond the scope of this book, having a general idea of how processors achieve instruction-level parallelism can be useful. We find that two lower bounds characterise the maximum performance of a program:

1. **Latency bound** - occurs when a series of operations must be performed serially because the output of one operation is required before the next can begin.
2. **Throughput bound** - characterises the raw computing capacity of the processor's functional units. This is the ultimate limit on program performance.

## Overall Operation
Figure 5.11 shows a very simplified view of a microprocessor:

![](_attachments/Screenshot%202023-11-13%20at%2019.15.20.png)

Modern processors are described as being **superscalar**, meaning:

* They can perform multiple operations on every clock cycle;
* They can perform machine code ops *out of order*

The overall design has two main parts:

* The **instruction control unit (ICU)** - reads instructions from memory and generates a set of primitive ops to perform.
* The **execution unit (EU)** - executes these ops.

The ICU reads instructions from an **instruction cache**. In general, the ICU fetches well ahead of currently executing instructions. One problem is that when a program hits a **branch** (i.e., a conditional [jump instruction](Control#Jump%20Instructions), there are two directions the program can go. Modern processors use **branch prediction** to mitigate this. They then use **speculative execution**, in which the processor begins fetching and decoding instructions along the branch it thinks will be taken, and even begins executing these operations, before it has been determined whether or not the branch prediction was correct.
The block labeled **fetch control** incorporates branch prediction.

The **instruction decode** unit takes actual program instructions and converts them into a set of primitive **operations** (sometimes called **micro-operations**). Each of these operations performs some simple task, such as:

* Adding two numbers
* Reading from memory
* Writing to memory

For CISC architectures, an instruction can be decoded into multiple operations. The details of how this is done are considered highly proprietary. This allows instructions to be divided among a set of dedicated hardware units, which can then execute different parts of multiple instructions in parallel.

The EU receives operations from the **instruction fetch unit**. These are dispatched to a set of **functional units** that perform the actual operations. These functional units are specialised to handle different types of operations.

Reading and writing memory is implemented by the load and store units. 

With speculative execution, operations are evaluated but the final results are not stored in registers or data memory until the program can be certain that these instructions should have actually been executed. Branch operations are sent to the EU, not to determine where the branch should go, but to determine whether they were predicted correctly.

Figure 5.11 indicates that different functional units are designed to perform different operations. For our example, the Intel Core i7 Haswell machine has eight functional units. Here is a partial list of each one's capabilities:

![](_attachments/Screenshot%202023-11-13%20at%2019.32.31.png)

We can see that this combination of functional units has the potential to perform multiple operations of the same type simultaneously. It has four units that can do integer ops, two that can do load ops, and two that can do floating-point ops. We'll see later that this impacts the maximum performance our program can achieve.

Within the ICU, the **retirement unit** keeps track of the ongoing processing and makes sure it obeys the sequential semantics of the machine-level program. Our figure shows a **register file** as part of the retirement unit, since the retirement unit controls the updating of these registers. Once the operations for an instruction have been completed and any branch prediction points leading to the instruction are confirmed, the instruction can be **retired**, meaning updates to the program registers are made. Otherwise, the instruction will be **flushed**, discarding any results that may have been computed.

> [!INFO]
> There was a bit more detail here, which I skipped.

## Functional Unit Performance
Figure 5.12 documents the performance of some of the arithmetic operations for our Intel Core i7 Haswell machine:

![](_attachments/Screenshot%202023-11-13%20at%2019.41.51.png)

Each operation is characterised by its:

* Latency
* Issue Time
* Capacity (number of functional units capable of performing that operation)

We see that the latencies increase in going from integer to floating point operations. Addition and multiplication ops *all* have issue times of 1, meaning that on each clock cycle, the processor can start a new one of these operations. This short issue time even holds for ops with latencies > 1, which is achieved through the use of **pipelining**.  A pipelined function is implemented as a series of **stages**, each of which performs part of the operation:

> [!WARNING]
> **FUN-FACT ALERT**
> A typical floating-point adder contains three stages (hence the three-cycle latency): one to process the exponent values, one to add the fractions, and one to round the result.

The arithmetic operations can proceed through the stages in close succession rather than waiting for one operation to complete before the next begins. This capacity can only be exploited if there are successive, logically independent operations to be performed.

> [!INFO]
> I skipped the rest of the info in this section.

## An Abstract Model of Processor Operation

