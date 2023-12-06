The difference between references and [Pointers](Educative%20Course.md#Pointers) in C++ is a little confusing. I thought I'd make some notes here on their similarities and differences.

### Basics
A reference can be thought of as an alias for another variable. Doing:

```cpp
type &newName = existingName;
```

declares a reference called `newName`.

### Differences with Pointers

[This StackOverflow discussion](https://stackoverflow.com/questions/57483/what-are-the-differences-between-a-pointer-variable-and-a-reference-variable) was really helpful here.

1. A pointer can be re-assigned. A reference cannot, and must be bound at initialisation:

```cpp
int x = 5;
int y = 6;
int &q; // error
int &r = x;
```

2. You can have nested pointers offering extra levels of indirection. References only offer one level of indirection; references to references are not allowed by the language syntax.
3. A pointer can be assigned `nullptr`, whereas a reference must be bound to an existing object.
4. A pointer needs to be dereferenced with `*` to access the memory location it points to. A reference can be used directly.
5. References cannot be put into an array.

The reason for the **1st point** is simply that, since references were introduced to serve as aliases to existing variables, they're always intended to refer to the same object after their initialisation.

Let's think about the **5th point** some more. When creating an array, we do memory allocation of a bunch of junk memory, and then assign its values to be what we want. As such, an array requires that its objets can be assigned. Since references can't be assigned, they can't belong to an array.

In modern C++, you can use `std::reference_wrapper`. This wraps a reference in a copyable, assignable object. It is used as a way to create store references inside standard containers (like `std::vector`):

```cpp
#include <functional>

int a = 5;
int b = 10;

std::reference_wrapper<int> refArray[2] = {a, b};
```

This creates an array of references to `int` objects.

### It's Kinda OK to Think of References as Pointers
A reference stores the address of a variable, as illustrated:

![](_attachments/Screenshot%202023-09-04%20at%2021.29.41.png)

and in many compiler implementations, a reference is indeed implemented as a **constant pointer**:

```cpp
\\ When you have
int x = 10;
int& ref = x;

\\ This might be represented as
int x = 10;
int* const ptr = &x; \\ ptr cannot be changed to point to a different address
```





