
In order to run the `hello` program, the individual C statements must be translated by other programs into low-level **machine-language** instructions. These are then packaged into an **exectuable object program** (aka **exectuable**) and stored as a binary disk file.

On a Unix system, the translation from source file to object file is performed by a **compiler driver**:

```bash
gcc -o hello hello.c
```

Here, the GCC compiler driver reads the source file `hello.c` and translates it into an executable object file `hello` . The translation is performed in the sequence of four phases shown in Figure 1.3. The programs that perform the four phases (**preprocessor, compiler, assembler, ** and **linker**) are known collectively as the **compilation system**.

![](_attachments/Screenshot%202022-05-13%20at%2011.09.58.png)

The purpose of each stage is as follows:
* *Preprocessing phase.* This modifies the original C program according to the directives that begin with the '#' character. For instance, line 1 of `hello.c` tells the preprocessor to read the contents of the system header file `stdio.h` and insert it directly into the program text. The result is another C program, typically with the `.i` suffix.
* *Compilation phase.* This translates the text file `hello.i` into the text file `hello.s`, which contains an *assembly-language program*. This includes the following definition of the function `main`:
![](_attachments/Screenshot%202022-05-13%20at%2011.17.02.png)
	Each of lines 2-7 describes one low-level machine-language instruction in a textual form. Assembly language is useful as it provides a common output language for different compilers for different high-level languages.
* *Assembly phase.* Next, the assembler translates `hello.s` into machine-language instructions, packages them into a form known as a **relocatable object program** and stores the result in the object file `hello.o`. This file is a binary file containing 17 bytes to encode the instructions for the function `main`. If viewed with a text editor, it would appear to be gibberish.
* *Linking phase.* Notice that our `hello`  program calls the `printf` function. This is part of the **standard C library** provided by every C compiler. The `printf` function resides in a separate precompiled object called `printf.o`, which must somehow be merged with our `hello.o` program. The linker handles this merging. The result is the `hello` file, which is an exectuable that is ready to be loaded into memory and exxecuted by the system.