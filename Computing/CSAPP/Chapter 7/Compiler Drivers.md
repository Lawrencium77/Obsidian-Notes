Consider the C program below. It will serve as a simple example, used in the rest of this chapter:

![](_attachments/Screenshot%202023-10-26%20at%2019.14.34.png)

Most compilation systems provide a **compiler driver**. This is a program that concatenates and manages the compilation process. It invokes the preprocessor, compiler, assembler, and linker as needed. To invoke the `gcc` driver, we type the following command into our shell:

```
linux> gcc -0 prog main.c sum.c
```

Figure 7.2 summarises the activities of the driver as it translates the example program into an executable object file:

![](_attachments/Screenshot%202023-10-26%20at%2019.19.21.png)

To run the executable `prog`, we simply do:

```
linux> ./prog
```

The shell invokes a function in the OS called the [loader](https://en.wikipedia.org/wiki/Loader_(computing)), which copies the code and data in contained within `prog` into memory (or [memory maps](../Chapter%209/Memory%20Mapping.md) them) , and then transfers control to the beginning of the program. ^727a6f



