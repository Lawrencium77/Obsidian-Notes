This section describes a running example that will be used through the rest of the chapter. It's based on the vector data structure shown in Figure 5.3:

![](_attachments/Screenshot%202023-11-10%20at%2016.49.20.png)

A vector is represented by a header plus some actual data:

![](_attachments/Screenshot%202023-11-10%20at%2016.49.43.png)

where `data_t` represents the data type of the vector. 

In our evaluation, we measure performance for `int`, `long`, `double`, and `float` data. We do so by compiling the program separately for different type definitions.

Figure 5.4 shows some basic procedures for generating vectors, accessing elements, and determining the length of a vector:

![](_attachments/Screenshot%202023-11-10%20at%2017.10.00.png)

These are just a bunch of helper functions, necessary when we write other code. As an optimisation example, Figure 5.5 performs a reduction operation on a vector:

![](_attachments/Screenshot%202023-11-10%20at%2017.11.03.png)

By using different definitions of compile-time constants `IDENT` and `OP`, the code can be recompiled to perform different operations. For instance:

```
#define IDENT 0
#define OP  +
```

sums the elements of a vector.

Through the rest of this chapter, we'll proceed through a series of transformations to the code, measuring performance as we do so.