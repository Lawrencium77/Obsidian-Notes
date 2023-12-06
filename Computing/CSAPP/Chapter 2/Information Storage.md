```toc
```

* Bytes are the smallest addressable unit of memory. 
* A machine-level program views memory as a large array of bytes, referred to as **virtual memory**. 
* Every byte is identified by a unique number, called its **address**
* The set of all possible addresses is called the **virtual address space**.
* This virtual address space is a conceptual image presented to the machine-level program. The actual implementation uses a combination of DRAM, flash memory, disk storage, special hardware, and OS software to provide the program with what appears to be a monolithic byte array.
* The value of a C pointer is the virtual address of the first byte of some block of storage. 
* The C compiler also associates **type** information with each pointer, so it can generate different machine code to access the data stored. 
* Although the C compiler maintains this type information, the actual *machine-level program it generates* has no information about data types.

#### Hexadecimal Notation

This was studied in *Petzold*, so there isn't too much to learn. It's worth noting that, in C, numeric constants starting with `0x` or `0X` are interpreted as hex. 

#### Data Sizes
* Every computer has a **word size**, indicating the size of pointer data (see [below](#^3e73dc)). 
* Since a virtual address is encoded by such a word, the most important system parameter determined by the word size is the maximum size of the virtual address space. For a machine with a $w$-bit word size, the program has access to $2^{w}$ bytes.
* A typical word size is 32/64 bits. For 32 bits, the virtual address space is ~4 GB. For 64-bits, it's 16 exabytes, or around $1.84 \times 10^{19}$ bytes.
* Most 64-bit machines can run programs compiled for 32-bit machines.
* The C language supports multiple data formats. Figure 2.3 shows the sizes for typical 32-bit and 64-bit programs.

![](_attachments/Screenshot%202022-05-26%20at%2008.33.01.png)

* Figure 2.3 shows that a pointer (type `char *`) uses the full word size.

> [!INFO]
> #### A Bit More on Word Size
> The word size of a machine determines more than just the size of a pointer. It reflects many aspects of a computer's design. Including:
> * Register size.
> * Size of pointer data.
> * Amount of data that can be transferred between the processor and L1 cache in a single op. In other words, it's **bus width** (but this is contentious).
> 
> Overall, it seems that word size is a fairly vague term and that people disagree on exactly what it means.
> 
> This info was gathered from [Wikipedia](https://en.wikipedia.org/wiki/Word_(computer_architecture)#Uses_of_words) and [StackOverflow](https://stackoverflow.com/questions/11472484/word-size-and-data-bus).
> 

^3e73dc

#### Addressing and Byte Ordering
* For program objects spanning multiple bytes, we must establish two conventions: what the address of the object will be, and how we will order the bytes in memory.
* A multi-byte object is stored as a contiguous sequence of bytes, with the address of the object given by the smallest address of the bytes used.
* For example, suppose an `int` has address `0x100`. Then the 4 bytes would be stored in memory locations  `0x100`,  `0x101`,  `0x102`, and  `0x103`.

* For the ordering of bytes representing an object, there are two common conventions: **little endian** and **big endian**. I already have some Anki cards on this. * An illustration (it considers a value of `0x01234567`):

![](_attachments/Screenshot%202022-05-26%20at%2008.52.24.png)

* For most programmers, the byte ordering used by their machine is invisible. But it can sometimes be an issue. 
* One way is when binary data are communicated over a network between machines that use different byte orderings. 
* Networking applications must follow established conventions for byte ordering to make sure the sending machine converts its internal representation to the network standard, while the receiving machine converts the network standard to its internal representation.

* The following figure shows C code that accesses and prints the byte representations of different program objects:

```C
#include <stdio.h>

typedef unsigned char *byte_pointer;

void show_bytes(byte_pointer start, size_t len) {
    int i;
	for (i=0;i<len;i++) 
		printf(" %.2x", start[i]);
    printf("\n");
}

void show_int(int x) {
    show_bytes((byte_pointer) &x, sizeof(int));
}

void show_float(float x) {
    show_bytes((byte_pointer) &x, sizeof(float));
}

void show_pointer(void *x) {
    show_bytes((byte_pointer) &x, sizeof(void *));
}

void test_show_bytes(int val) {
	int ival = val;
	float fval = (float) ival;
	int *pval = &ival;
	show_int(ival);
	show_float(fval);
	show_pointer(pval);
	}

int main(){
	test_show_bytes(12345);
}
```


* Procedures `show_int`, `show_float`, and `show_pointer` demonstrate how to use procedure `show_bytes` to print the byte representations of program objects. 
* Observe that they simple pass `show_bytes` a pointer `&x` to their argument `x`, casting the pointer to be of type `unsigned char *`.
* This cast indicates to the compiler that the program should consider the pointer to be a sequence of bytes rather than an object of the original data type. The pointer will then be to the lowest byte address occupied by the object.
* Our program has the following output:

![](_attachments/Screenshot%202022-07-06%20at%2023.27.09.png)

* Our argument 12,345 has representation`0b11000000111001= 0x00003039`. 
* We see that our machine (which is Linux 64) puts the least significant byte value of `0x39` first, indicating that it is little-endian.
* Note that the Linux 64 machine uses an 8-byte address.

#### Representing Strings
* A string in C is encoded by an array of characters terminated by the null (having $0$) character. 
* Each character is represented by some standard encoding, with the most common being the ASCII standard.
* If we run our routine `show_bytes` with arguments `"12345"` we get the result `31 32 33 34 35 00`. 
* Observe that the ASCII code for decimal digit `y` happens to be `0x3y`.

#### Representing Code
Consider the following C function:
```C
int sum(int x, int y) {
	return x + y;
}
```

* A fundamental concept of computer systems is that a program, from the perspective of the machine, is just a sequence of bytes. 
* The machine has no information about the original source program. 

#### Introduction to Boolean Algebra
* Some of the content here was covered in *Petzold*, but there are still useful ideas.
* One point of note is the the operations `~`, `&`, `|`, and `^` encode logical operations `NOT`, `AND`, `OR`, and `XOR` respectively. This is exactly the syntax used in C.
* We also extend Boolean operations to operate on **bit vectors**. We define these operations to be element-wise, i.e. $(a \& b)_i = a_i \& b_i$
* One application of bit vectors is to represent finite sets. We can encode any subset $A \subset \{0,1,...,w-1\}$ with a bit vector $[a_{w-1},...,a_1,a_0]$, where $a_i=1$ iff $i \in A$. 
* This means we write $a_0$ on the *right*. For example, the bit vector $b=[01101001]$ encodes the set $A = \{0,3,5,6\}$. 
* With this way of encoding sets, Boolean operations $|$ and $\&$ correspond to set union and intersections, respectively, and ~ corresponds to set complement.
* We'll see the encoding of sets by bit vectors in a number of practical applications. 
* For example, in Chapter 8, we'll see that there are a number of different *signals* that can interrupt the execution of a program. We can selectively enable or disable different signals by specifying a bit-vector mask, where a $1$ in bit position $i$ indicates that signal $i$ is disabled (and vice versa).

#### Bit-Level Operations in C
* One useful feature of C is that it supports bitwise Boolean operations. Some examples are:

![](_attachments/Screenshot%202022-07-09%20at%2010.54.51.png)

* The best way to determine the effect of a bit-level expression is to expand the hexadecimal arguments to their binary representations, perform the operations in binary, and then convert back to hexadecimal.
* A common use of bit-level operations is to implement *masking*. The operation `x & 0xFF` yields the least significant byte of x, but sets all other bytes to `0`. If `x = 0x89ABCDEF`, we'd get `0x000000EF`.

#### Logical Operations in C

* C also provides a set of logical operators, `||`, `&&`, and `!`. These correspond to `OR`, `AND` and `NOT`, respectively. 
* These can easily be confused with the bit-level operations, but their behaviour is different. 
* The logical operations treat any nonzero argument as representing `TRUE` and argument $0$ as being `FALSE`. They return either $1$ or $0$, indicating a result of either `TRUE` or `FALSE`.
* Some examples are :

![](_attachments/Screenshot%202022-07-09%20at%2011.16.21.png)

* A bitwise operation will have behaviour matching that of its logical counterpart only in the special case that the arguments are restricted to $0$ or $1$.
* Logical operators `&&` and `||` do not evaluate their second argument if the result of the expression can be determined by evaluating the first argument. For instance, the expression `a && 5/a` will never cause a division by zero. 
* This is similar to [short-circuiting](https://docs.python.org/3/library/stdtypes.html#boolean-operations-and-or-not) in Python.

#### Shift Operations in C
* C also provides a set of *shift* operations for shifting bit patterns to the left or right. 
* For an operand `x` having bit representation $[x_{w-1},x_{w-2},...,x_0]$, the C expression `x<<k` yields a value with bit representation $[x_{w-k-1},x_{w-k-2},...,x_0,0,...,0]$. 
* In other words, it shifts `x` by `k` bits to the left, dropping off the `k` most significant bits and replacing them with zeros.
* Shift operations associate from left to right, so `x<<j<<k` is equivalent to `(x<<j)<<k`.
* There is a corresponding right shift operation, but it has slightly subtle behaviour. Generally, machines support two types of right shift:

**Logical**: Fills the left end with $k$ zeros, giving $[0,...,0,x_{w-1}, x_{w-2},...,x_k]$.

**Arithmetic:** Fills the left end with $k$ repetitions of the most significant bit, yielding $[x_{w-1},...,x_{w-1},x_{w-1},x_{w-2},...,x_k]$. This might seem a bit odd, but it's useful for operating on signed integer data.

* Some examples of applying different shift operations are shown below:
![](_attachments/Screenshot%202022-07-09%20at%2012.53.19.png)
* In practice, almost all compilers assume arithmetic right shifts for signed data. For unsigned data, right shifts must be logical.
