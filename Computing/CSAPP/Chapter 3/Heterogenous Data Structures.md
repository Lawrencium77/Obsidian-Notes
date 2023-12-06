C provides two mechanisms for creating data types by combining objects of different types:

* **Structures**, declared using the keyword `struct`. They aggregate multiple objects into a single unit.
* **Unions**, declared using the keyword `union`. They allow an object to be referenced using several different types.

## Structures
The implementation of structs is similar to that of arrays in that all of the components of a structure are stored in a contiguous region of memory, and a pointer to a structure is the address of its first byte. The compiler maintains information about each structure type indicating the byte offset of each field. It generates references to structure elements using these offsets. 

As an example, consider this struct:

```C
struct rec{
	int i;
	int j;
	int a[2];
	int *p;
}
```

The structure contains four fields, of different types. In total, it is 24 bytes long:

![](_attachments/Screenshot%202023-10-08%20at%2019.01.52.png)

To access the fields of a struct, the compiler generates code that adds the appropriate offset to the address of its first element. For example, suppose variable `r` of type `struct rec*` is in register `%rdi`. Then the following code copies element `r->i` to element `r->j`:

![](_attachments/Screenshot%202023-10-08%20at%2019.03.55.png)

Since the offset of field `i` is 0, the address of this field is simply the value of `r`. To store into field `j`, the code adds offset 4 to the address of `r`.

To generate a pointer to an object within a struct, we can simply add the field's offset to the structure address. For instance, we can generate the pointer `&(r->a[1])` by adding offset $8 + 4 \cdot 1 = 12$. 

The key point is that the selection of different fields of a struct is handled completely at compile time. 

## Unions
Unions provide a way to circumvent the type system of C. They allow a single object to be referenced according to multiple types. The syntax of a union declaration is identical to that of structures, but its semantics are very different. 

Consider the following declarations:

```C
struct S3 {
    char c;
	int i[2];
	double v; 
};

union U3 {
    char c;
	int i[2];
	double v; 
};
```

In a struct, each field corresponds to a different block of memory. In a union, all members share the same memory location. A union will be as big as its largest member. If you were to assign a value to the `c` member of the union and then assign a value to the `v` member, the `c` value would be overwritten because they share the same memory block.

Unions can be useful in several contexts but can also lead to nasty bugs, since they bypass the type safety provided by C. One application is when we know in advance that the use of two different fields in a data structure will be mutually exclusive. Then, declaring these two fields as part of a union rather than a struct will reduce the total space allocated.

## Data Alignment
Many systems place restrictions on the allowable addresses for the primitive data types, requiring that the address for some objects must be a multiple of some value $K$ (typically 2, 4, or 8). These **alignment restrictions** simplify the design of the hardware forming the interface between the processor and the memory system. 

For example, suppose a processor always fetches 8 bytes from memory within an address that must be a multiple of 8. If we can guarantee that any `double` will be aligned to have its address be a multiple of 8, then the value can be read with a single read, or written with a single write. Otherwise, we may need to perform two memory accesses, since the object might be split across two 8-byte memory blocks.

x86-64 systems will work regardless of the data alignment. But Intel recommends that data be aligned to improve memory system performance. Their alignment rule is based on the principle that any primitive object of $K$ bytes must have an address that is a multiple of $K$:

![](_attachments/Screenshot%202023-10-08%20at%2019.23.33.png)

> [!INFO]
> [Wikipedia](https://en.wikipedia.org/wiki/Data_structure_alignment) refers to this as ***natural alignment***.

The compiler places directives in assembly code indicating the desired alignment for global data. For example, the assembly-code declaration of the jump table seen on p271 contains this directive:

```
.align 8
```

This ensures that the data following it (in this case, the jump table) will start with an address that is a multiple of 8. Since each table entry is 8 bytes long, the successive elements will obey the 8-byte alignment restriction.

For code involving structures, the compiler may need to insert gaps in the field allocation to ensure that each structure element satisfies its alignment requirement. For instance, consider this struct:

```C
struct S1 {
	int i;
	char c;
	int j;
}
```

Suppose the compiler uses the minimal 9-byte allocation, diagrammed as follows:

![](_attachments/Screenshot%202023-10-08%20at%2019.27.37.png)

It would be impossible to satisfy the 4-byte alignment requirement for both fields `i` and `j`. Instead, the compiler inserts a 3-byte gap between fields `c` and `j`:

![](_attachments/Screenshot%202023-10-08%20at%2019.28.18.png)

As a result, `j` has offset 8, and the overall structure is 12 bytes. 
In addition, the compiler may need to add padding to the end of the structure so that each element in an array of structures will satisfy its alignment requirement. For example, this struct:

```C
struct S2 {
	int i;
	int j;
	char c;
}
```

satisfies all its alignment requirements, but:

```C
struct S2 d[4];
```

would provide a scenario where it's not possible to satisfy the alignment requirements for each element of `d`. Instead, the compiler allocates 12 bytes for struct `S2`, with the final 3 bytes being padding:

![](_attachments/Screenshot%202023-10-08%20at%2019.31.39.png)





