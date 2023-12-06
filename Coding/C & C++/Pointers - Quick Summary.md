I thought it useful to make a condensed summary of pointers. 

```toc
```

#### What They Are

A pointer is a variable that stores the memory address of another variable:

![](_attachments/Screenshot%202023-07-29%20at%2011.32.28.png)

(In the above diagram, I've drawn word-sized cells of virtual memory).

#### Creating a Pointer

To declare a pointer, we do something like:

```C
int *p;    // pointer to an integer
double *d; // pointer to a double
```

#### Address-Of Operator
The `&` operator can be used to get the address of a variable:

```C
int x = 10;
int *p;
p = &x;  // p now holds the address of x
```

#### Dereferencing Pointers
This means **getting the value stored at an address**. The `*` is used to do this:

```C
int x = 10;
int *p;
p = &x;     // p now holds the address of x
int y = *p; // y is now 10
```

#### Pointer to a Pointer
A pointer can contain the address of another pointer. This creates a *chain* of pointers. Consider:

```C
int x = 10;
int *p;
int **pp;
p = &x;
pp = &p;
```

Diagrammatically:

![](_attachments/Screenshot%202023-07-29%20at%2011.38.51.png)

#### Size of a Pointer
The size of pointer data is given by the [word size](Information%20Storage#^3e73dc) of your machine.