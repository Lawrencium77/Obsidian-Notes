These notes were made using the [educative course](https://www.educative.io/courses/learn-c-from-scratch).

# Still to do:
- [ ] Understand variadic functions syntax

# Contents
```toc
```

# Section 1: Why use C?

## What is C?
One thing that is worth taking away is that with few exceptions, all (procedural) programming languages are essentially the same, but:

1. Names of standard functions are different
2. Rules of syntax are different
3. Functionality included in standard libraries is different
4. APIs are different and use different names for things

Once you know how to program in one language, all you have to do is learn some different syntactic rules, and learn the names of the various functions that you will be using, and learn what APIs provide the needed functionality. No big deal.

## When to use C?
The interpreter speed in C is far quicker than in other languages:

![](_attachments/Screenshot%202022-07-19%20at%2008.59.24.png)

This can sometime be very useful. For instance, if you have a data processing operation, and you know it will take a long time to run, then C can improve the speed by an order of magnitude (or more).

A useful rule of thumb is that interpreted languages should be used for prototyping, and C for performance.

# Section 2: Basic Types and Operators

## Data Types and Sizes
There are 4 basic data types in C. Their meaning, and sizes, are as follows:

![](_attachments/Screenshot%202022-07-19%20at%2009.04.10.png)

There are also qualifiers `short`, `long`, `signed`, and `unsigned` that can be applied to these basic types:

![](_attachments/Screenshot%202022-07-19%20at%2009.05.27.png)

## Constants
An important **character constant** to know about is the constant `\0`. This represents that character with value zero, sometimes called the `NULL` character. It is used to terminate variable-length strings.

**String constants** can be specified using double quotes (""). They are technically an array of characters that is terminated by a null character `\0` at the end. This means that the storage required to represent a string of length $n$ is actually $n+1$.

### Enumeration Constants
An enumeration constant is a list of constant integer values that you can assign to arbitrary labels. They provide a convenient way to associate constant values with names. For example, we could store the days of the week like this:

```C
enum days { MON, TUE, WED, THU, FRI, SAT, SUN }
```

In this instance, `MON` will be assigned value `0`, `TUE` will be assigned value `1`, etc. We don't use strings to represent such quantities because in C strings are slightly clunky to work with. Comparing two strings is not as easy as typing `if (day == "MON")`. It requires a call to a function in `string.h` called `strcmp()`.
Another reason is that because `enum` data types are represented as integers, you can do integer operations on them.

### `const` keyword
The `const` keyword is used to specify that a variable's value cannot be changed. It tells the compiler to prevent the user from changing the value of that variable. For instance:

```C
const int x = 33;
```

## Declarations
Unlike Python, which is a **dynamically typed** language, C is a **statically typed** language. From a practical point of view, this means that in C we have to declare, up front, the **type** of every variable we use.

## Expressions
The following binary arithmetic operators can be used in C: `+`, `-`, `*`, `/` and the modulus operator `%`.

The relational operators are `>`, `>=`, `<` and `<=`. There are also two equality operators: `==` and `!=`.

Lastly, there are two **logical operators** `&&` (logical AND) and `||` (logical OR). By default in C, the results of relational and logical operators are evaluated to integers values: `0` for FALSE and `1` for TRUE.

## Type Conversions
There are two kinds of way to convert one data type to another: **implicit** type conversion, and **explicit** type conversion.

### Implicit Type Conversion
The operators we have looked at can deal with different types. For instance, we can apply the `+` operator to an `int` as well as a `double`. There are rules in C that govern how operators convert different types, to evaluate the results of the expressions.

For example, when a floating-point number is assigned to an integer value in C, the decimal portion of the number gets truncated. On the other hand, when an integer value is assigned to a floating-point variable, the decimal is assumed as `.0`.

This sort of implicit or automatic conversion can produce nasty bugs, especially for example when performing multiplication or division using mixed types, e.g. integer and floating point values. Here is some example code illustrating these points:

![](_attachments/Screenshot%202022-07-20%20at%2008.45.08.png)

### Explicit Type Conversion
There is a mechanism in C to perform **type casting**. This forces an expression to be converted to a particular type of our choosing. We surround the desired type in brackets and place that before the expression to be coerced. For example:

![](_attachments/Screenshot%202022-07-20%20at%2008.50.15.png)
prints `a` as a `float`.

### String Conversion Library Functions
There are some built-in functions in C to perform some basic conversions between strings and numeric types. There are two useful functions to now about converting ASCII strings to numeric types: `atoi()` (ASCII to integer) and `atof` (ASCII to float). We need to `#include` the library `stdlib.h` to use these functions.

Converting numeric types to strings is a bit more tricky. First, we have to allocate space in memory to store the string. Then we use the `sprintf()` built-in function to "print" the numeric type in our string.

Here is some example code that converts strings to numerics, and vice versa:

![](_attachments/Screenshot%202022-07-20%20at%2008.58.51.png)

## Defining your own types with `typedef`
You can assign an alternate name to a data type. This `typedef` statement allows you to do this.

For instance, we can use it to define a type called "counter" which is an alternate name for an integer. We can then declare variables to be of type "Counter"

```C
typedef int Counter;
Counter i, j, k;
```

`Typedef` isn't used too often in most basic C code, but is used in applications requiring a high degree of portability. New types may be defined for basic variables and typedef may be used in header files to tailor the program to the target machine.

Somewhere you see `typedef` used more often is while simplifying the declaration of compound types, such as the `struct` type (which we'll see later).

## Increment and Decrement Operators
Pretty simple. The `++` and `--` operators add `1` and subtract `1` respectively. 

One thing to note is that you can use these two operators either before or after the operand, e.g. `a++` and `++a`. When the operator is used before the operand it is called a **prefix** operator. When the operand is used after, it's called a **postfix** operator.

Prefix operators cause the change to happen **before** the value is used, and postfix operators cause the change to happen **after**. Here is a concrete example:

![](_attachments/Screenshot%202022-07-20%20at%2021.34.59.png)
The output being:

![](_attachments/Screenshot%202022-07-20%20at%2021.35.39.png)

It's interesting that we do the increment operation *independently* of the addition. 

It could be argued that these operators are unnecessarily confusing. There is a risk of misusing them, so if we want to increment by 1 then we can do so explicitly as well, e.g. `x = x +1`.

# Section 3: Control Flow
## Loops
### For Loop
Generic for-loop expressions look like this:

![](_attachments/Screenshot%202022-07-20%20at%2021.50.33.png)
### While Loop
The while-loop syntax is:

![](_attachments/Screenshot%202022-07-20%20at%2021.51.44.png)

### Do-While Loop
This is another version of a while-loop that is essentially the same as a while-loop, but reverses the order of the `program_statements` and `conditional_expression`. This guarantees that the `program_statements` run at least once:

![](_attachments/Screenshot%202022-07-20%20at%2021.54.33.png)
## Conditionals 
### If Statement
The basic if statement looks like:

![](_attachments/Screenshot%202022-07-20%20at%2021.55.30.png)

### Else
Syntax is:

![](_attachments/Screenshot%202022-07-20%20at%2021.56.05.png)

### Else If

Syntax is:

![](_attachments/Screenshot%202022-07-20%20at%2021.56.25.png)

### The Conditional Operator
There is a convenient operator for simple if-else constructs, using the **conditional operator**:

```C
condition ? expression1 : expression2
```

It enables you to shorten your code a bit.  This basically says: "If `condition` is `True`, do `expression1`, `else` do `expression2`".

E.g.

```C
int age = 23;
(age>=18)? (printf("Can Vote")) : (printf("Can't vote"));
```

## Switch 
The case of multiple if-else statements strung together to test for different values, and execute different code, is common enough that there is a special construct called the **switch statement** that is provided for this case:

![](_attachments/Screenshot%202022-07-20%20at%2022.01.07.png)

This is equivalent to the following series of if-else statements:

![](_attachments/Screenshot%202022-07-20%20at%2022.01.33.png)

The decision to use a switch statement versus a series of if-else statements is purely stylistic.

## Break & Continue
The `break` and `continue` keywords in C work in exactly the same way as Python:

# Section 4: Functions
## Defining a Function
The syntax for defining a function is as follows:

![](_attachments/Screenshot%202022-07-23%20at%2009.55.48.png)

It's worth remembering that we can define a function that doesn't return anything using `void`.

## Argument Checking
Note that if we pass an argument of the wrong type to a function, the program may still compile, and even run. But it will spit out some crazy values.

We need to be very careful that the input values we pass to functions, and the output values we receive, are what we expect.

## Variable Scope

^731284

Any variables declared inside a function are **local to that function**, and are not accessible outside of it. 

Likewise, code within a function doesn't have access to variables that have been declared outside of that function. If we want this functionality, you have to specify a **global** variable. Any variable declared outside of **any** function (including `main()`) is global, and can be used by every function. In C, global variables are known as **external** variables.

For example, in the following code, the variable `myGlob` is global:

![](_attachments/Screenshot%202022-07-23%20at%2010.03.43.png)

For more info, see [these notes](../Python/Scope%20in%20Python/Scope%20Types.md) I made on variable scope in Python.

## Automatic vs Static Variables
Each time a function is called by another piece of code, all the local variables for that function are created (that is, memory is allocated to them). When a function is finished, all of that local memory storage is de-allocated, and those variables essentially disappear. These are known as **automatic** local variables.

If you want local variables to persist, you can declare them as **static** local variables. You simply insert the word `static` in front of the variable type when you declare it inside your function. When declared in this way, the variable will **not be destroyed** when the function exits. Next time the function is called, the variable will have retained the value from the previous function call. **It's a sort of global variable, but one that is still only accessible within the function in which it's declared.**

In this example, the function `myFun()` records how many times it has been called:

![](_attachments/Screenshot%202022-07-23%20at%2010.10.28.png)

Static variables can improve efficiency. For instance, if your function declares a **large** local variable whose values don't change from one function to the next, it may be more efficient to declare it as static, so that it's initialised only once.

## Variadic Functions
A **variadic** function is one which accepts a variable number of input arguments. In Python, we can write these using `*args` or `**kwargs`. 

# Section 5: Complex Data Types
## Arrays 
We can do an array declaration as follows:

```C
int grades[5];
```

This declaration says we want to allocate space in memory for 5 integer values, and we can refer to that block in memory with the variable name "grades". Note that we have not initialised any values of the array.

We can examine the contents of an array by **indexing** into it:

![](_attachments/Screenshot%202022-07-23%20at%2015.38.05.png)

In this case, the values of the array will contain whatever values happened to be in memory at those locations before.

Once an array of a given size is declared (and the memory allocated), the array size is fixed. You cannot extend the array (or make it smaller). We will see exceptions to this when talking about dynamically allocated memory using `malloc()` but for now, assume arrays are fixed in size once declared.

If we try to access an element of an array beyond its bounds, like for example accessing the 6th element of `grades`, **sometimes C will not prevent us from doing that**. We **won't even get a warning from the compiler**.

To understand why this is not nonsensical, we have to understand some details about how C represents arrays in memory, and how accessing memory via indexing works. When we declare a variable `grades` that's a 5-element array of integers, C does two (main) things:

1. A contiguous block of memory is reserved. The amount that's set aside is equal to the number of elements multiplied by the size of the declared element type. So if we declare `int grades[5];`, then 20 bytes (5 $\times$ 4) is set aside.
2. The variable name `grades` is assigned to a **pointer** to the address in memory corresponding to the first element of the array.

When you then index into the `grades` array, C will look at the appropriate place in memory (defined by the `grades` pointer), plus the appropriate number of "steps" into the memory block, as defined by the index and the size of the basic type as defined in the array declaration.

This means that if we ask for the 500th element of the array, C simply reads the memory location 500 $\times$ 4 $=$ 2000 bytes past the beginning of the memory block, whatever that may be.

This lack of "protection" is a prime cause of programming errors that can be difficult to debug. What's particularly dangerous, however, is that this feature of C also applies to **assigning** values to an array.

## Array Initialisation and Updating Array Elements

We can assign a value to the $i$-th element of an array by doing:

```C
array[i] = value;
```

As mentioned above, we can ask C to assign values to elements beyond the bounds of the array and **C won't complain**. These locations in memory may be used to store other important things! They may correspond to some other variable in your program, or some information that's not even part of your program at all. For instance, some part of the OS that is controlling the hard disk, or the fans, etc.

In general, we can initialise the values of an array using `{}` like this:

```C
int grades[5] = {4, 3, 2, 5, 1}
```

We can initialise specific values (and not others) like this:

```C
int grades[5] = {[0]=1, [2]=3, [4]=5}
```

**All other elements** are set to zero.

## Multidimensional Arrays
Here is how we define a 3-D array of integers, initialise values, and index into it:

![](_attachments/Screenshot%202022-07-23%20at%2015.55.50.png)

The output here would be:

![](_attachments/Screenshot%202022-07-23%20at%2015.56.13.png)

This allows us to understand the pattern in which C assigns values to the array elements from a simple list.

### Matrix Calculations

If we wish to do lots of matrix calculation, it is best to make use of a pre-existing API. Two common choices are:

1. The GNU Scientific Library [Vectors and Matrices](https://www.gnu.org/software/gsl/doc/html/)
2. [LAPACK](https://netlib.org/lapack/)(Linear Algebra PACKage)

## Variable-Length Arrays
In modern C (the C99 standard and above), it is possible to declare arrays without knowing their size until run-time. This can be done as follows:

![](_attachments/Screenshot%202022-07-23%20at%2018.17.53.png)

The array `grades` has a size that is determined inside  `main()`. 

## Command-Line Arguments
The `main()` function can be defined using two arguments:

1. `int argc`
2. `char *argv[]`

When your program is executed, the first argument `argc` will contain the **number of command-line arguments passed, plus one**. The first argument is always the name of your program. 

The second argument `argv` is a 1D array of strings. As we will see later, the `(char *)` data type, which is actually a pointer to a character, is typically used in C to represent strings (as an array of characters).

It's important to note that **all command-line arguments are treated as null-terminated strings**. At least initially. So if we want to pass integers or floating-point values, we will have to convert them from strings in our code.

In the previous section's example, we use the `atoi()` function to convert **ascii** to an **integer**. Here is another example that does nothing but output the number of input arguments, and their value:

![](_attachments/Screenshot%202022-07-23%20at%2018.23.18.png)

## Structures
Structures are another way of grouping together different elements of data, into one named variable. Unlike arrays, in which all elements have to be of the same type, in structures, you can store any combination of any data types you want. Structures can even contain other structures.

Here's what a structure definition looks like:

![](_attachments/Screenshot%202022-07-23%20at%2018.27.17.png)

Here is how we would use a structure in a program:

![](_attachments/Screenshot%202022-07-23%20at%2018.27.43.png)

We can use `typedef` to shorten how we declare structure variables. It allows us to avoid having to use the `struct` keyword every time we declare a new variable that is our structure type:

![](_attachments/Screenshot%202022-07-23%20at%2018.30.04.png)

# Section 6: Memory - Stack vs Heap
The way we have been declaring variables so far has been putting variables on the **stack** in C.

## The Stack

### What is the Stack?
It's a region of memory that stores temporary variables created by each function (including `main()`). It operates a LIFO data structure, that is managed and optimised by the CPU quite closely.

Every time a function declares a new variable, it is "pushed" onto the stack. Then every time a function exits, **all** of the variables pushed onto the stack by that function are freed (i.e. deleted). 

### Memory Management with the Stack

The advantage of using the stack to store variables is that memory is managed for you. You don't have to allocate memory by hand, or free it once you don't need it anymore. The CPU organises stack memory very efficiently, so reading from and writing to stack variables is very fast.

### Stack Variables are Local in Nature
Again, when a function exists, all of its variables are popped off the stack. Thus, stack variables are **local** in nature. This is related to the concept of **variable scope** that we discussed previously. A common bug in C programming is attempting to access a variable that was created on the stack inside some function, from a place in your program that's outside the function.

### Limitation
Depending on your OS, there is a limit on the size of variables that can be stored on the stack. This is not the case for variables allocated on the **heap**.

### Summary
* The stack grows and shrinks as functions push and pop variables;
* There is no need to manage the memory yourself. Variables are allocated and freed automatically;
* The stack has size limits (on variable size);
* Stack variables only exist while the function that created them is running.

## The Heap
### What is the Heap?
The heap is a region of your computer's memory that is not managed automatically for you, and is not as tightly managed by the CPU. It is a more free-floating region of memory (and is larger).

Unlike the stack, the heap does not have size restrictions on variable size (apart from the physical limitations of your computer).

Also, unlike the stack, variables created on the heap are accessible by any function, anywhere in your program. Heap variables are essentially global in scope.

However, it is slower to read from and write to because we have to use **pointers** to access memory on the heap.

### Memory Leak
To allocate memory on the heap, you must use `malloc()` or `calloc()`, which are built-in C functions. Once you have allocated memory on the heap, you are responsible for using `free()` to deallocate that memory once you don't need it anymore. If you fail to do so, your program will have what is known as a **memory leak**. That is, memory on the heap won't be available to other processes (the idea being that we have "leaked" some memory away).

As we will see in the debugging section, there is a tool called `valgrind` that can help detect memory leaks.

## Stack vs Heap: Pros and Cons

![](_attachments/Screenshot%202022-07-23%20at%2019.37.13.png)

## Examples on Stack and Heap
Here's a short program that creates its variables on the **stack**. It looks like the other programs we've seen so far:

![](_attachments/Screenshot%202022-07-23%20at%2019.47.41.png)

As a side note, there is a way to tell C to keep a stack variable around, even after its creator function exits. We use the `static` keyword when declaring the variable. A variable with the `static` thus becomes something like a global variable, but one that is only visible inside the function that created it. It's a strange construction - one that you are unlikely to need except for specific circumstances.

Here's another version of this program that allocates all of its variables on the **heap** instead of the stack:

![](_attachments/Screenshot%202022-07-23%20at%2019.50.49.png)

As we can see, using `malloc()` to allocate memory and `free()` to deallocate it is no big deal, but is a bit cumbersome. There are also a bunch of pointers all over the place now. The `malloc()`, `calloc()`, and `free()` functions deal with **pointers** instead of actual values. We will talk more about pointers shortly; the bottom line is that pointers are a special data type in C that stores **addresses in memory** instead of actual values.

## When to use the Heap/Stack?
Use the heap when:

* You need to allocate a large block of memory, and you need to keep that variable around a long time (e.g. a global variable).
* You need variables like arrays and structs that can change size dynamically.

Use the stack whenL

* You are dealing with relatively small variables that are local in scope.

We will talk about dynamically allocated data structures after we talk about pointers.

# Section 7: Pointers
## Pointers
Pointers allow you to work with dynamically allocated memory blocks. They are used to manipulate the heap. 

They are a special data type that contains an **address** to a data in memory. It's not unlike the concept of a phone number, or house address.

Their purpose is to allow you to manually and directly access a block of memory. They're used a lot for **strings** and **structs**. It's not difficult to imagine that passing the address of a large block of memory to a function is more efficient that making a copy of it and passing it that copy, only to delete the copy when your function is done with it. This is known as **passing by reference** (pointers) vs **passing by value** (standard).

### Declaring a Pointer
The syntax for declaring a pointer `p` of type `T` is

```C
T *p;
```

We often assign a value to pointers using the **address of** operator

```C
p = &age;
```

### Dereferencing a Pointer
In the above example, the variable `p` is the address of an `int`, whereas `*p` is the value of the `int` that p points to. Accessing the value that a pointer points to is called **dereferencing** a pointer.

Pointers are 8 bytes in size. This means a pointer can hold 32 bits, corresponding to $2^{32}$ distinct addresses in memory. So 32-bit pointers allow us to access up to 4 GB of RAM. On 64-bit systems, pointers are 8 bytes, corresponding to $2^{64}$ addresses in memory. It will be a **long time** before your computer has that much RAM (1 billion GB).

## Pointers and Arrays
Another powerful aspect of pointers is that they allow us to communicate with arrays.

When you declare an array using an expression like `int vec[5];`, what is really happening is that a block of memory is allocated to store 5 integers, and the `vec` variable is a pointer that points to the first element in the array. When you index into the array, C is using **pointer arithmetic** to step into the array and read the value at the corresponding memory address.

The explicit (but never used) way of doing this would be to use `malloc()` or `calloc()` to allocate the array (in this case on the heap) and then use pointer arithmetic to read off the (e.g.) 3rd value:

![](_attachments/Screenshot%202022-07-24%20at%2018.33.20.png)

The expression `*(vec+2)` essentially says to go to the location of that `vec`  points to, and take two steps.

## Passing Pointers to a Struct
### Pointer to a Struct
Pointers can also be used to point to a struct. This would be done like:

![](_attachments/Screenshot%202022-07-24%20at%2018.36.44.png)

On lines 21-23 we see the more common shorthand for using pointers with structs. The point is that the notation

```C
p->day = 15;
```

sets the field `day` of the struct at location `p` to value `15`.

## Passing Pointers to Functions
A very important concept here is that of passing function arguments by reference, which allows us to alter a variable outside the function scope.

### Pointers to Functions
We can use a pointer to point to a function. You can then pass this function pointer to other functions as an argument, store it in a struct, etc. Here's a small example:

![](_attachments/Screenshot%202022-07-24%20at%2018.42.07.png)

The first argument to the function `doMath` is:

```C
int (*fn)(int a, int b)
```

This is a **pointer to a function**. The `(*fn)` says this is a pointer to a function, and we shall refer to that function as `fn`. The subsequent `(int a, int b)` says it's a function that takes two `int` arguments as inputs.

On lines 25-27, we simply pass the **name** of one of the functions we defined above. 

### Functions Arguments: Passing by Value vs Passing by Reference
Here is some code illustrating passing by value. **Passing by value** is essentially the default:

![](_attachments/Screenshot%202022-07-24%20at%2018.47.43.png)

We could instead pass by reference:

![](_attachments/Screenshot%202022-07-24%20at%2018.48.49.png)

We write `myFun()` such that it takes an input argument that is a pointer to an int. In `main()`, we pass it the *address* of `y` instead of its value.

## Dynamically Allocated Memory
<u>TL;DR</u>: In the heap, we can control the amount of memory allocated to certain variables with built in C functions that we discuss below.

Once you have allocated a variable on the stack, it is fixed in size. In contrast, if you use `malloc()` or `calloc()` to allocate an array on the heap, you can use `realloc()` to resize it at some later time. In order to use these, we must `#include <stdlib.h>` .

The life cycle of a heap variable involves three stages:

1. allocating the heap variable using `malloc()` or `calloc()`
2. (optionally) resizing the heap variable using `realloc()`
3. releasing the memory from the heap using `free()`

### Allocating Memory with `malloc()` or `calloc()`
The `malloc()` function takes as input the size of the memory block to be allocated. The `calloc()` function is like `malloc()` except that it initialises all elements to zero. The `calloc()` function takes two input arguments: the number of elements, and the size of each element.

Here's an example of how we might use `malloc()` to allocate memory to hold an array of 10 structs:

```C
#include <stdio.h>

typedef struct {
	int year;
	int month;
	int day
} date;

int main() {
	date *mylist = malloc(sizeof(data) * 10)
	
	\\ Rest of code 

	return 0;
}
```

### Resizing a variable using  `realloc()`
Let's say we use `calloc()` to allocate an array of 3 floating-point numbers, and later in the program we want to increase this to 5. This could be done like:

```C
double *vec = calloc(3, sizeof(double));
vec = realloc(vec, sizeof(double)*5);
```

The `realloc` function takes two arguments: the variable to resize, and the new desired size.

### Freeing a Memory Block using `free()`
Any memory that has been allocated is **reserved** - it can't be used until it's deallocated with `free()`. If you fail to do so, we have what's called a **memory leak**. If your program uses a lot of heap memory that isn't deallocated, then our program may slow down or crash.

The rule is that for every `malloc()` or `calloc()`, there **must** be a corresponding `free()`.

The syntax of the `free()` function is pretty simple. For a pointer `T *p`, we deallocate the associated memory with:

```C
free(p);
```

# Section 8: Strings

## Basics
Being honest, C is slightly awkward for dealing with character strings.

Strings are actually, under the hood, an **array of characters**. Single character variables are declared using single-quotation marks. Multi-character strings use double quotation marks.

String variables are declared as on line 8 below. The double square brackets signify an array. Just like arrays, we can access string elements through indexing.

![](_attachments/Screenshot%202022-08-07%20at%2020.29.41.png)

### Null-termination
An important thing to remember about strings is that they are always **null-terminated**. This means that the last element of the character array is a "null" character, abbreviated `\0`. When you declare a string as in line 8 above, you don't put the null termination character yourself - the compiler does it for you. 

### Modifying Strings
In C, it is a bit tricky to modify a declared string.

Importantly, once a string is declared to be a given length, you cannot just make it longer or shorter by reassigning a new constant to the variable.

We can edit the contents of a string though, since it is simply an array of characters.

![](_attachments/Screenshot%202022-08-07%20at%2020.33.42.png)

To summarise: in C, strings are simply **null-terminated arrays of characters**.

## String Handling Routines in the C Standard Library
The C standard library allows us to manage strings in a few ways. It also allows conversion between strings and numbers

### Comparing Strings
You cannot use the `==` operator to test whether two strings are equal. You have to use a special string handling function, since it has to do a "deep" comparison, comparing each element against each other. 
This is done with the `strcmp` function.

### Converting Strings to and From Numeric Types
There are several functions for doing this:

* `double atof(s)` converts the string pointed to by `s` into a floating-point number, returning the result.
* `int atoi(s)` converts the string `s` into an integer.

There are a host of others.

### Numbers to Strings
The common way of converting a numeric type - like an integer or a float - to a string is with the `sprintf()` function. It's much like `printf()`, but instead of printing to the screen, it "prints" to a character string.

## Arrays of Strings
Strings are just arrays of characters, terminated by a null character. The variables that hold strings are actually **pointers** to the head of the array. We can therefore use an array of pointers to store an array of strings. Here is an example:

![](_attachments/Screenshot%202022-08-07%20at%2020.52.49.png)

This is a useful reminder that the syntax `char *provinces[]` creates an *array of pointers*.

#  Section 9: I/O Streams

## Basics
The standard C library includes a handful of functions for performing input and output in C. C an UNIX both makes use of **streams**. We can distinguish between two types of streams: text and binary.

Text streams consist of lines, where each line has zero or more characters, and is terminated by a new line character, `\n`. Binary streams are more "raw" and consist of a stream of any number of bytes.

C programs have all three of the standard streams built in. When your program uses `printf()` to print to the screen, you are using **stdout**.

Files are associated with streams, and we will see below how to read/write from them.

### Character I/O: `getchar()` and `putchar()`
When you want to read data a single character at a time, you can use `getchar()`. The output function `putchar()` will write one character at a time to stdout.

![](_attachments/Screenshot%202022-08-07%20at%2021.15.48.png)

### Formatted I/O: `printf()` and `scanf()`.
In general, the `printf()` function takes two arguments: first, a string argument that indicates what to write to the screen; and second, a list of variables that provide the data to the various elements in the formatted string.

To read formatted data from standard input, we can use `scanf()`. Just like `printf()`, it takes as a first argument a format string, followed by other arguments specifying the destination of each argument. It requires that these destination arguments be **addresses** of the relevant locations in memory (we can feed it a pointer, for example).

Here is an example in which we read from stdin a date:

![](_attachments/Screenshot%202022-08-07%20at%2021.19.29.png)

The `scanf()` function ignores blanks and tabs in the format string, and it skips over white space as it looks for input values.

## Input and Output with Files
C also lets us read and write data to files using `fopen()` and `fclose()`.

### Opening and Closing Files with `fopen()` and `fclose()`
Before a file can be read or written to, it has to be **opened** using `fopen()`. This takes as arguments a string corresponding to the filename, and a second  argument (also a string) corresponding to the **mode**. The mode is read, write or append. `fopen()` then returns a pointer to the (open) file. After reading and/or writing to your file, you will need to **close** it using `fclose()`.

### Reading and Writing to Files
There are many functions in `stdio.h` for reading from and writing to files. There is a collection of functions for reading and writing ASCII data, as well as binary data.

#### ASCII Files
There are functions to read single characters at a time, (`getc()` and `putc()`), functions to read and write formatted output (`fscanf()` and `fprintf()`), and there are functions to read and write single lines at a time (`fgets()` and `fputs()`).

See the course notes for more info.

## Binary Files
There are many circumstances in which you may want to read from and write to binary files. Advantages of binary files over ASCII files are as follows:

* They're typically smaller in size
* They can be read from and written to faster (no need to convert between raw bytes and ASCII characters)

Their disadvantage is that they're not human-readable.

The `fread()` and `fwrite()` functions are used to read and write binary data from and to binary files. 

The course notes have more info. The key point is that as long as you know what the binary **format** is (that is, how many bytes represent each value), then you can read and write them in "raw" binary using `fread()` and `fwrite`.

#  Section 10: Macros and the C Preprocessor
## The C Preprocessor and the `#define` Statement
### The C Preprocessor
The C Preprocessor is a part of the C compilation process that recognises special statements, analyses them (before compilation) and acts on them as required.

This the first section of the compilation system, as described in CSAPP (page 41).

It can be used to make programs more efficient, readable, and portable.

### The `#define` Statement
You can use the `#define` statement to assign symbolic names to program constants. For example:

![](_attachments/Screenshot%202022-08-07%20at%2021.37.41.png)
These are all valid uses of `#define`. These statements are **not** like variable assignment - no memory is allocated. Remember, the preprocessor acts **before** compilation. What is actually happening is that the preprocessor is going through the C code that follows, and simply substituting each instance of the symbolic name with the associated value. It's basically a search-and-replace.

The `#define` statements can appear anywhere in the program, although common convention is to put them at the top of source code files.

## Macros
You can use `#define` statements in more advanced ways, which are sometimes called **macros**. A macro is typically used to define something that takes one or more arguments.

For example, you could define a macro to perform the square of a number:

![](_attachments/Screenshot%202022-08-09%20at%2008.56.03.png)


### Variadic Macros
A variadic macro is one with a variable number of arguments. Here is an example:

![](_attachments/Screenshot%202022-08-09%20at%2008.57.55.png)

There are countless other examples of things you can do with macros.

We can also tell the compiler how we want to execute macros by using **conditional statements**. 

## Conditional Compilation
The preprocessor also facilitates conditional compilation.

One can use `#ifdef`, `#endif`, `#else` and `#ifndef` statements to achieve conditional compilation - for example to create one program that can be compiled and run on different computer systems.

Similarly, one can use `#if`, `#elif` preprocessor statements to define macros differently depending on the value of other `#define` values. We won't get into this, but it's worth being aware of.

# Section 11: Compiling, Linking, Makefile, Header Files
## Splitting a Program into Multiple Files
It is a general practice to divide your code into different categories or files. This section will teach you how to do so correctly.

The programs we've seen so far have all been stored in a single source file. 

We saw in the section on functions that one way of including custom-written functions in your C code is to simply place them in your main source file, above the `main()` declaration. A better way to re-use functions is to place them in their own file, and to include a statement above `main()` to **include** that file.

When compiled, it's just like copying and pasting the code above `main()`. But for the purposes of editing and writing code, it allows you to keep things in separate files. 

### Header Files

> [!INFO]
> I think I have a decent description of header files in some of my [CSAPP notes](Symbol%20Resolution#More%20Background%20-%20How%20External%20Symbols%20are%20Used%20in%20C/C++).

A common convention in C programs is to write a **header file** (with **.h** suffix) for each source file (**.c** suffix) that you link to your main source code. The logic is that the source file contains all of the code, and the header file contains the function **prototypes** - that is, just a declaration of which functions can be found in the source file.

An important point to remember is that as long as we can look at the header file, we have all the information about what functions are defined in the source file, and how they are used (what their inputs/outputs are). If we want to examine the functions closer, we can look at the source file. The header file can be thought of as an **interface** between the source code and the programmer.



### The `#include` Statement
The `#include` statement is used to link a header file or a library with a source file. We've already seen the `#include <stdio.h>` example.

For any source file, the template for using the `#include` statement is:

`#include <file name>` or `#include "file name"`

See the next section for the difference between these two templates.

In order to include external code in your C program (code that is located in a separate file), two things need to happen:

1. The external C source code is passed to the compiler;
2. Your own C program has access to the function prototypes associated with the external code (via the header file).

These two points make sense when looking at the examples given in the course notes.

### Search Path
The compiler looks in several places for header files that you include with the `#include` directive, depending on how you use it. If you use include with angled brackets (`#include <stdio.h>`) then the compiler will look in a series of "default" system-wide locations. If you use double quotes (`#include "neuron.h"`), then the compiler will look in the directory containing the current file. 

It's possible to add other directories to the search path by using the `Idir` compiler option, where `dir` is the other directory. You might have to do this if you link your code to an external C library that is not part of the standard C library, and does not reside in the usual "system" default locations.

## The GNU `make` utility and Makefiles
There is a UNIX tool called `make` that is commonly used to compile C programs that are made up of several files, and (sometimes) involve several compilation steps. There is a lot of power in the `make` tool, but here we introduce just a simple use of it that lets us avoid having to remember a long, complicated compile command.

The `make` utility uses a special plain-text file that you write that has to reside in the same directory as your program, and must be called `Makefile`. You can think of a `Makefile` as a **recipe** for making your program (i.e. linking and compiling).

Suppose we have a source file `primes.c` that we wish to use to calculate primes. A simple Makefile might look like:

```C
go: go.c primes.c 
	gcc -o go go.c primes.c
```

The first word and colon `go:` on line 1 represents the **name** of a recipe called **go**. The list of files after the colon represent all of the things that go **depends upon**. On the next line, there is a tab, followed by a compile command. This represents the steps that are required to "make" the "go" recipe.

Now all we have to do from the command-line is type `make`, and make will "make" the recipe for "go". 

### Introducing More Generalisation to the Makefile
There are a number of features of a Makefile we can utilize to make the whole idea more useful. We can introduce "macros" (like a variable) to generalize the name of the C compiler to use, the flags to pass the compiler, the location of any library files, etc. Here is what a more generalized Makefile might look like for our primes example from above:

```C
CC = gcc 
CFLAGS = -Wall 
DEPS = primes.h 
OBJ = go.o primes.o 

%.o: %.c $(DEPS) 
		$(CC) $(CFLAGS) -c -o $@ $< 
		
go: $(OBJ) 
		gcc $(CFLAGS) -o $@ $^
```

We have moved all the specific details into the macros on the top, and and what remains below in the rules themselves is expressed in terms of those macros. 

Note that we have a term in the `CLFAGS` macro that looks like this: `-Wall`. This is a flag to the compiler to turn on all Warnings. There are many warnings that the compiler will tell you about - such as variables that are never used, uninitialised variables, etc.

Here is what it looks lke when we run `make` using `Makefile2.txt`:

![](_attachments/Screenshot%202022-10-08%20at%2017.21.46.png)

You can see that three commands end up being run. The neat thing is that if we type `make` again, we get this:

![](_attachments/Screenshot%202022-10-08%20at%2017.22.15.png)

We are told that the "go" rule is update to date. The `make` program checks to see which files have changed since the last make, and only executes the rule(s) (if any) that need to be re-done.

In the long run, using Makefiles is a good idea because:
* It's faster to recompile things (less typing, and it only recompiles based on what's changed);
* It organizes all the steps in a (potentially complex) compilation into one place. This makes it easier for other people to compile your code.

The next chaper in this course deals with the GNU Project Debugger. 







