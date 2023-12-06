Machine code provides two low-level mechanisms for implementing condition behaviour: it tests data values, and then alters either the:

* **control flow**,  or 
* the **data flow** 

based on the results of these tests.

**Control** flow is more general, and more common. 
Normally, machine code instructions are executed **sequentially**, in the order that they appear in the program. But the execution order can be modified with a **jump** instruction, indicating that control should pass to some other part of the program. The C compiler must generate instruction sequences that build upon this low-level mechanism to implement the control constructs of C.

This section first covers the two ways of implementing condition operations. We then describe methods for presenting loops and switch statements.

```toc
```

## Condition Codes
In addition to its [general-purpose registers](Accessing%20Information.md), the CPU maintains a set of **single-bit condition-code registers**. These describe attributes of the most recent [arithmetic/logical operation](Arithmetic%20and%20Logical%20Operations.md). They can then be tested to perform conditional branches. These condition codes are the most useful:

* `CF`: Carry flag. Whether the most recent operation generated a carry out of the most significant bit. Used to detect overflow for unsigned values.
* `ZF`: Zero flag. Whether the most recent operation yielded zero.
* `SF`: Sign flag. Whether the most recent operation yielded a negative value.
* `OF`: Overflow flag. The most recent operation caused a two's-complement overflow - either negative or positive.

> [!INFO]
> In the x86-64 architecture, the condition codes refer to specific bits within the **RFLAGS** register. Not all 64 bits of this register are used for condition codes. The remaining bits are reserved for other purposes.

For example, suppose we do an `ADD` to perform the C assignment `t = a + b`, where `a, b, t` are integers. Then the condition codes would be set according to:

```
CF (unsigned) t < (unsigned) a
ZF (t == 0)
SF (t < 0)
OF (a < 0 == b < 0) && (t < 0 != a < 0)
```

With the exception of `leaq`, all logical/arithmetic instructions cause the condition codes to be set. In addition to these, there are two instruction classes (having 8-, 16-, 32-, and 64-bit forms) that set the condition codes without altering any other registers. These are listed in Figure 3.13:

![](_attachments/Screenshot%202023-10-03%20at%2009.23.01.png)

The `CMP` instructions set condition codes according to the differences of their operands. They behave in the same way as `SUB` instructions, except they don't update their destinations. `CMP` instructions set the zero flag if their two operands are equal. 
The `TEST` instructions behave in the same way as `AND` instructions (i.e. a bitwise `AND`) except that, again, they set condition codes without altering their destinations. Typically, the same operand is repeated, or one of the operands is a mask indicating which bits should be tested.

## Accessing the Condition Codes
Rather than reading the condition codes directly, there are three common ways to use them:

1. Set a single byte to `0` or `1` depending on some combination of them;
2. Conditionally jump to another part of the program;
3. Conditionally transfer data.

For the first case, the instructions in Figure 3.14 set a single byte depending on some combination of the condition codes. We refer to this entire class as the `SET` instructions:

![](_attachments/Screenshot%202023-10-03%20at%2009.34.22.png)

A `SET` instruction has either one of the low-order single-byte register elements, or a single-byte memory location, as its destination. To generate a 32-bit or 64-bit result, we must also clear the high-order bits. A typical instruction sequence to compute the C expression `a < b`, where `a, b` have type `long`, is:

![](_attachments/Screenshot%202023-10-03%20at%2009.38.18.png)

And for reference:

![](_attachments/Screenshot%202023-10-03%20at%2009.39.04.png)

Note that, although all arithmetic and logical operations set the condition codes, the above *descriptions* of them a specific to the case where an instruction performing `t = a - b` has just been executed.

> [!INFO]
> It's important to note that machine code doesn't distinguish between signed and unsigned values. Unlike in C, it doesn't associate a data type with each program value. Some circumstances require different instructions to handle signed and unsigned operations. It is the responsibility of the compiler to choose these instructions.

## Jump Instructions
A **jump** instruction can cause execution to switch to a completely different position in the program. Jump destinations are generally indicated in assembly code by a **label**. Consider this example:

![](_attachments/Screenshot%202023-10-03%20at%2009.47.04.png)

The instruction `jmp .L1` will cause the program to skip the `movq` instruction and instead resume execute with the `popq` instruction. In generating the executable, the assembler determines the addresses of all labeled instructions and encodes the **jump targets** (the addresses of the destination instructions) as part of the jump instructions.

Some jump instructions are **conditional** - they only execute depending on some combination of the condition codes. Examples of such instructions are shown in Figure 3.15:

![](_attachments/Screenshot%202023-10-03%20at%2020.48.23.png)

## Jump Instruction Encodings
For the most part, the detailed format of machine code will be beyond our scope. However, understanding how the targets of jump instructions are encoded will become important when studying linking in Chapter 7.

Jump targets are written using symbolic labels. The assembler, and later the linker, generate the proper encodings of the jump targets. There are several different encodings for jumps, but the most commonly used ones are **PC relative**. That is, they encode the difference between the address of the target instruction and the address of the instruction immediately followed by the jump. A second encoding method is to give an **absolute** address. 

Below is an example of PC-relative addressing. It contains two jumps: the `jmp` instruction on line 2 jumps forward to a higher address, while the `jg` instruction on line 7 jumps back to a lower one:

![](_attachments/Screenshot%202023-10-03%20at%2020.52.45.png)

The diassembled version of the `.o` format generated by the assembler is as follows:

![](_attachments/Screenshot%202023-10-03%20at%2020.53.10.png)

The annotations on the RHS are generated by the disassembler. The jump targets are indicated as `0x8` and `0x5`. However, looking at the byte encodings of the instructions, we see that the target of the first jump instruction is encoded as `0x03`. This difference is due to the relative encoding; adding this to `0x05` - the address of the next instruction - we get `0x08`, the address of the instruction on line 4.

> [!INFO]
> It's important to appreciate that the choice of PC relative of absolute addresses is made by the assembler.

These examples also illustrate that the value of the program counter when performing PC-relative addressing is the address of the instruction following the jump, not that of the jump itself. 

The following shows the disassembled version of the program after linking:

![](_attachments/Screenshot%202023-10-03%20at%2020.59.15.png)

The instructions have been relocated to different addresses, but the encodings of the jump targets in lines 2 and 5 remain unchanged. By using a PC-relative encoding of the jump targets, we therefore get two benefits:

1. The instruction can be compactly encoded;
2. The object code can be shifted to different positions in memory without alteration (useful when linking).

## Implementing Conditional Branches with Conditional Control
The most general way to translate conditional expressions from C into machine code is to use a combination of (conditional) jumps. (As an alternative, we will see that some conditionals can be implemented by conditional transfers of data rather than control). 

Figure 3.16(a) shows the C code for a function that computes the absolute value of the different of two numbers. It also has the side-effect of incrementing one of two counters, `lt_cnt` or `gt_cnt`.  `gcc` generates the assembly code in 3.16(c). 

![](_attachments/Screenshot%202023-10-03%20at%2021.03.07.png)

As this example demonstrates, the general form of an if-else statement in C is:

```
if (test-expr)
	then-statemenet
else
	else-statement
```

For this general form, an assembly implementation typically adheres to the following form, where we use C syntax to describe the control flow:

```
	t = test-expr
	if (!t)
		goto false;
	then-statement
	goto done
false:
	else-statement
done:
```

## Implementing Conditional Branches with Conditional Moves
The conventional way to implement conditional operations is through a conditional transfer of control, as above. This mechanism is simple and general, but can be very inefficient on modern processors.

An alternate strategy is through a conditional transfer of ***data***. This approach computes both outcomes of a conditional operation and then selects one based on whether or not some condition holds. This makes sense only in restricted cases, but can be implemented by a simple *conditional move* instruction that is better matched to the performance characteristics of modern processors.

Figure 3.17(a) shows an example of C code that can be compiled using a conditional move. The function computes the absolute value of its arguments `x` and `y`, as did our earlier example (Figure 3.16). 

![](_attachments/Screenshot%202023-10-03%20at%2021.13.13.png)

For this function, `gcc` generates the assembly code in Figure 3.17(c), having an approximate form shown by the C function `cmovdiff` shown in Figure 3.16(b). 

To understand why code based on conditional data transfers (Figure 3.17) can outperform code based on conditional control transfers (Figure 3.16), we must understand a little about how modern processors operate. As we'll see in Chapters 4 and 5, processors achieve high performance through **pipelining**, where an instruction is processed via a sequence of stages. Each one performs one small portion of the required operation (e.g. fetching the instruction from memory, determining the instruction type, reading from memory, performing an arithmetic operation, writing to memory, and updating the PC). This approach achieves high performance by overlapping the steps of successive operations, such as fetching one instruction while performing arithmetic operations for a previous instruction. 

Doing this requires being able to determine the sequence of instructions to be executed well ahead of time in order to keep the pipeline full. When the machine encounters a conditional jump, it cannot determine which way the branch will go until it has evaluated the branch condition. Processors employ sophisticated **branch prediction logic** to try to guess whether each jump instruction will be followed. As long as it can guess reliably (modern microprocessor designs try to achieve success rates of ~90%), the instruction pipeline will be kept full of instructions. But mispredicting a jump requires the processor to discard much of the work it has already done, and then begin filling the pipeline with instructions starting at the correct location. ^2b0cbb

Unlike conditional jumps, the processor can execute conditional move instructions without having to predict the outcome of some test. It simply reads the source value, checks the condition code, and then either updates the destination register or keeps it the same. 

To understand how conditional operations can be implemented via conditional data transfers, consider the following general form of a conditional expression and assignment:

```
v = test-expr ? then-expr : else-expr
```

The standard way to compile this using conditional control transfer would be:

```
	if (!test-expr)
		goto false;
	v = then-expr;
	goto done;
false:
	v = else-expr
done:
```

For code based on a conditional move, we have:

```
v = then-expr;
ve = else-expr;
t = test-expr;
if (!t) v = ve;
```

The final statement is implemented with a conditional move - `ve` is copied to `v` only if `t` does not hold.

Not all conditional expressions can be compiled using conditional moves. Most significantly the abstract code we've shown evaluates both `then-expr` and `else-expr` regardless of the test outcome. If one of those two expressions could possibly generate an error condition, this could lead to invalid behaviour. This is the case for our earlier example (Figure 3.16).

Using conditional moves does not always improve code efficiency. For example, if either the `then-expr` or `else-expr` evaluation requires a significant computation, then this effort is wasted when the corresponding condition does not hold. **Compilers must take into account the relative performance of wasted computation versus the potential for performance penalty due to branch mis-prediction**. In truth, they do not really have enough information to make this decision reliably - they are imperfect!

Overall, conditional data transfers offer an alternative strategy to conditional control transfers for implementing conditional ops. They can only be used in restricted cases, but these cases are fairly common and provide a better match to the operation of modern processors.

## Loops
C provides several looping constructs - `while`, `do-while`, and `for`. No corresponding instructions exist in machine code. Instead, combinations of conditional tests and jumps are used to implement the effects of loops. We study the translation of loops as a progression, starting with `do-while` and then working towards more complex implementations.

### Do-While Loops
The general form of a `do-while` loop is:

```C
do
	{body-statement}
	while (test-expr);
```

They are very similar to `while` loops, except that the `body-statement` is executed at least once. This general form can be translated into conditionals and `goto` statements as:

```C
loop:
	body-statement
	t = test-expr;
	if (t)
		goto loop;
```

Figure 3.19 shows an example C program, with corresponding assembly-language code:

![](_attachments/Screenshot%202023-10-07%20at%2020.18.08.png)

### While Loops
There are a number of ways to translate a `while` loop into machine code. Two of them are used in code generated by `gcc`. Both use the same loop structure as we saw for `do-while` loops, but differ in how to implement the initial test.

The first translation method, which we call ***jump to middle*** is shown below:

```
	goto test;
loop:
	body-statement
test:
	t = test-expr;
	if (t)
		goto loop
```

The second translation method, which we call ***guarded do***, first transforms the code into a `do-while` loop:

```
t = test-expr
if (!t)
	goto done;
do 
	body-statement
	while (test-expr);
done:
```

In other words, we perform our test once before entering the `do-while` loop. This can be transformed into `goto` code as:

```
t = test-expr;
if (!t)
	goto done;
loop:
	body-statement
	t = test-expr;
	if (t)
		goto loop;
done:
```

Using this implementation, the compiler can often optimise the initial test, for example, determining the test condition will always hold.

### For Loops
The general form of a `for` loop

```C
for (init-expr; test-expr; update-expr)
	body-statement
```

can be written using a `while` loop:

```C
init-expr;
while (test-expr) {
	body-statement
	update-expr
}
```

The code generated by `gcc` for a `for` loop then follows one of our two translation strategies for `while` loops. 

Overall, we see that all three forms of loops in C can be translated by a simple strategy, generating code that contains one or more conditional branches. Conditional transfer of control provides the basic mechanism for translating loops into machine code.

## Switch Statements
A `switch` statement provides a multiway branching capability based on the value of an integer index. As an example:

```C
void switch_eg(long x, long n,  long *dest) {
	long val = x;
	switch (n) {
		
		case 100:
			val *= 13;
			break;

		case 102:
		    val += 10;
		    break
	}
	*dest = val;
}
```

`switch` statements allow an efficient implementation using a data structure called a **jump table**. This is an array where entry $i$ is the address of a code segment implementing the action the program should take when the switch index equals $i$.[^fn1] This is preferable to a long series of `if-else` statements, since the time complexity in performing this lookup is independent of the number of switch cases. 

`gcc` selects the method of translating a `switch` statement based on the number of cases and the sparsity of the case values. Jump tables are used when there are a number of cases, and they span a small range of values.

The following figure shows an example of a `switch` statement, and an alternative version that corresponds closely to the output assembly code. It has a number of interesting features, including case labels that don't span a contiguous range, cases with multiple labels, and cases that *fall through* to other cases:

![](_attachments/Screenshot%202023-10-07%20at%2020.40.32.png)

Figure 3.23 shows the assembly code generated when compiling `switch_eg`:

![](_attachments/Screenshot%202023-10-07%20at%2020.42.13.png)

The compiler first shifts the range to between 0 and 6 by subtracting 100 from `n`. There are then five distinct locations to jump to, based on the value of `index`. Each of these labels identifies a block of code implementing one of the case branches. 
The key step in executing a `switch` statement is to access a code location through the jump table. This happens in line 16 of the C code, and line 5 of the assembly code.

In the assembly code, the jump table is indicated by the following declarations: ^585231

![](_attachments/Screenshot%202023-10-07%20at%2020.49.33.png)

The key point of this section is that the use of a jump table allows a very efficient way to implement a multi-way branch. In our case, the program could branch to five distinct locations with a single jump table reference.


[^fn1]: To be clear: the value of the switch index is used to index into the jump table.





