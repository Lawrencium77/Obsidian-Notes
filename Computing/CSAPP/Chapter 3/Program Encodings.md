We can compile C programs using:

```
linux> gcc -Og -o p file1.c file2.c
```

The option `-Og` instructs the compiler to apply a level of optimisation such that the machine code follows the overall structure of the original C code. We will use this as a learning tool. Higher levels of optimisation can be specified with `-O1`, `-O2`, and [others](https://chat.openai.com/share/52aaf9e7-ccef-4ed5-b35e-493a7e9f5f5a).

The `gcc` command invokes a whole [sequence](../Chapter%201/Programs%20are%20Translated%20by%20Other%20Programs%20into%20Different%20Forms.md) of programs to turn source code into executable code.

## Machine-Level Code
Machine instructions describe the behaviour of a program as if each instruction is executed in sequence. In practice, the processor hardware is more elaborate, executing many instructions concurrently. 

In the compilation process, it's the compiler that does most of the work. It transforms arbitrary C code into assembly, which is very close to machine code. Its main feature is that it's more readable.

Parts of processor state are visible in machine code that are normally hidden from the C programmer:

* **Program counter***
* **Register file** - 16 named locations storing some values
* The condition code registers hold status information about the most recently executed instruction. These are used to implement control flow, such as `if` and `while` statements.
* A set of vector registers can each hold one or more integers or floats.

The following C program:

```C
long mult2(long, long);

void multstore(long x, long y, long *dest) {
		 long t = mult2(x, y);
		 *dest = t;
}
```

yields assembly code containing the following:

```
multstore:
	pushq   %rbx
	movq    %rdx, %rbx
	call    mult2
	movq    %rax, (%rbx)
	popq    %rbx
	ret
```

The `pushq` instruction indicates that the contents of register `%rbx` should be pushed onto the program stack. 

To inspect the contents of **machine-code** files, a class of programs called **disassemblers** can be invaluable. These generate a format similar to assembly code from machine code. With Linux systems, the program `objdump` can serve this role given the `-d` command-line flag:

```
linux> objdump -d output.o
```

> [!INFO]
> The authors give more info on how to interpret the disassembler output. But I skipped for now.









