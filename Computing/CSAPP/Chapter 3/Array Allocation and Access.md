## Basic Principles
For data type `T` and integer constant `N`, consider a declaration of the form:

```C
T A[N];
```

Let us denote the starting location as $x_A$. The declaration has two effects:

1. It allocates a contiguous region of $L\cdot N$ bytes, where $L$ is the size (in bytes) of data type $T$.
2. It introduces an identifier $A$ that can be used as a pointer to the beginning of the array.

The conversion between array access in C and assembly code is quite straightforward. For example, suppose `E` is an array of values of type `int` and we wish to evaluate `E[i]`, where the address of `E` is stored in register `%rdx` and $i$ is stored in register `%rcx`. Then the instruction

```
movl (%rdx,%rcx,4),%eax
```

will perform address computation $x_E + 4i$, read that memory location, and copy the result to register `%eax`. 

## Pointer Arithmetic 
C allows arithmetic on pointers. If a pointer `p` has type `T`, and its value is $x_p$, then the expression `p + i` has value $x_p + L\cdot i$, where $L$ is the size of data type `T`.

As an example, suppose the starting address of integer array `E` and integer index $i$ are stored in registers `%rdx` and `%rcx` respectively. The following are some expressions involving `E`:

![](_attachments/Screenshot%202023-10-08%20at%2018.39.55.png)

## Nested Arrays
The array elements of nested arrays are arranged in **row-major** order. To access elements of multidimensional arrays, the compiler generates code to compute the offset of the desired element element and then uses the relevant `MOV` instruction. In general, for an array declared as:

```C
T D[R][C];
```

array element `D[i][j]` is at memory address:

$$
\& \mathrm{D}[\mathrm{i}][\mathrm{j}]=x_{\mathrm{D}}+L(C \cdot i+j)
$$

## Fixed-Size Arrays
This section describes some of the optimisations made by the C compiler when operating on multi-dimensional arrays. I didn't look at it in too much detail.

## Variable-Size Arrays
Historically, C only supported multidimensional arrays where the sizes could be determined at compile time. Programmers requiring variable-size arrays had to allocate storage for these arrays using functions such as `malloc` or `calloc`, and then explicitly encode the mapping of multidimensional arrays into single-dimension ones via row-major indexing.

ISO C99 introduced the capability of having array dimension expressions that are computed as the array is being allocated. We declare an array:

```C
int A[expr1][expr2]
```

where the dimensions of the array are determined by evaluating `expr1` and `expr2` at the time the declaration is encountered. For example, we can write a function to access element $i,j$ of an $n\times n$ array as:

```C
int var_ele(long n, int A[n][n], long i, long j) {
    return A[i][j];
}
```

`Gcc` generates code for this function as:

![](_attachments/Screenshot%202023-10-08%20at%2018.51.37.png)

We see that referencing variable-size arrays requires only a slight generalisation over fixed-size ones. The dynamic version must use a multiplication instruction to scale $i$ by $n$, rather than a series of shifts and adds. In some processors, this multiplication can incur a significant performance penalty, but is unavoidable.


