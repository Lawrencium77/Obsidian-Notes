```toc
```

## Background

The linker resolves symbol references by associating each reference with **exactly one** symbol definition from the symbol tables of its input relocatable object files. 

Symbol resolution is straightforward for references to symbols that are defined in the same module as the reference, and is usually done at compile time. The compiler allows only one definition of each local symbol per module. The compiler also ensures that static local variables have unique names.

However, resolving references to global symbols is trickier. When the compiler encounters a symbol that isn't defined in the current module, it assumes it's defined in some other module, generates a linker symbol table entry, and leaves it for the linker to handle. If the linker is unable to find a definition for the referenced symbol in any of its input modules, it throws an error. For instance, if we try to compile and link the following source file on a Linux machine:

```C
void foo(void);

int main() {
	foo();
	return 0;
}
```

then the compiler runs fine, but the linker terminates when it cannot resolve the reference to `foo()`. 

Symbol resolution for global symbols is also tricky because multiple object modules can define global symbols with the same name. The solution adopted by Linux systems involves cooperation between the compiler, assembler, and linker in order to somehow choose one of the definitions and discard the rest.

## More Background - How External Symbols are Used in C/C++
I added this section to clarify what it actually looks like to use external symbols in C/C++. When you use external symbol in your source code, it involves:

1. Declaring the symbol without providing its definition;
2. Using the linker to find its definition from another object file.

Here's a breakdown:

#### 1. External Symbol Declaration
This is as simple as:

```C
int some_external_variable;
void some_external_function();
```

#### 2. Header Files
For ease of use, these external declarations are placed in header files, which we then `#include` to bring into the source file:

```
#include "library_header.h"
```

This explains why header files are so important - they bunch together all the symbol declarations that we wish to use elsewhere.

#### 3. Linking
When you compile your source code, the compiler produces object files that have unresolved references for these external symbols. The linker resolves these.

> [!INFO]
> #### Aside: Mangling of Linker Symbols in C++
> Both C++ and Java allow overloaded methods that have the same name in the source code but different arguments. In order for the linker to tell the difference between these different overloaded functions, the compiler encodes each unique method and argument list combo into a unique name. This process is called **mangling**. 
> 
> In fact, there are many more reasons for name mangling. [Wikipedia](https://en.wikipedia.org/wiki/Name_mangling) has a good description of more.

## How Linkers Resolve Duplicate Symbol Names
This section describes the approach taken by Linux compilation systems. 

At compile time, the compiler exports each global symbol to the assembler as either **strong** or **weak**. The assembler encodes this information implicitly in the symbol table of the relocatable object file. Functions and initialised global variables get strong symbols. Uninitialised global variables get weak symbols.

Given this notation, Linux linkers use the following rules for dealing with duplicate symbol names:

1. Multiple strong symbols with the same name are not allowed.
2. Given a strong symbol and $\geq 1$ weak symbol with the same name, use the strong symbol.
3. Given multiple weak symbols with the same name, choose any one of them.

For example, suppose we try to compile and link these two modules:

![](_attachments/Screenshot%202023-10-26%20at%2020.37.09.png)

The symbol `x` is defined twice, but uninitialised in the second case. Hence the linker will quietly choose the strong symbol defined in the first case. 

The application of rules 2 and 3 can introduce some insidious run-time bugs, especially if the symbol definitions have different types. When in doubt, invoke the linker with a flag such as the GCC `-fno-common` flag, which triggers an error if it encounters multiply-defined global symbols. Or use the `-Werror` options, which turns all warnings into errors.

## Linking with Static Libraries

So far, we have assumed that the linker reads a collection of relocatable object files and links them together into an output executable file. In practice, all compilation systems provide a mechanism for packaging related object modules into a single file called a **static library**, which can then be used as input to the linker. When it builds the output executable, the **linker copies only the object modules in the library that are referenced by the application program**. 

This last part is important - it means we don't have to copy the entire shared library in the any executable the depends upon it.

On Linux systems, static libraries are stored on disk in a particular file format known as an **archive**. An archive is a concatenation of relocatable object files, with a header that describes the size and location of each member object file. Archive filenames have the `.a` suffix.

To make our discussion concrete, consider the pair of vector routines in Figure 7.6:

![](_attachments/Screenshot%202023-10-26%20at%2020.49.37.png)

Each routine, defined in its own object module, performs a vector operation on an array. To create a static library of the functions, we'd use the `ar` tool:

```
linux> gcc -c addvec.c multvec.c
linux> ar rcs libvector.a addvec.o multvec.o
```

To use the library, we might write an application such as `main2.c` in Figure 7.7:

![](_attachments/Screenshot%202023-10-26%20at%2020.51.22.png)

The header file `vector.h` defines the function prototypes for the routines in `libvector.a`. To build the executable, we would compile and link the input files `main2.0` and `libvector.a`:

```
linux> gcc -c main2.c
linux> gcc -static -o prog2c main2.o ./libvector.a
```

Figure 7.8 summarises the activity of the linker:

![](_attachments/Screenshot%202023-10-26%20at%2020.53.23.png)

The `-static` argument tells the compiler driver that the linker should build a fully linked executable object file that can be loaded into memory and run without any further linking at load time. 
When the linker runs, it determines that the `addvec` symbol defined by `addvec.o` is referenced by `main2.o`, so it copies `addvec.o` into the executable. Since the program doesn't reference any symbols defined by `multvec.o`, the linker does **not** copy this module into the executable. 

> [!SUMMARY]
> Static libraries offer two main benefits:
> 1. Selective inclusion  - only the necessary object modules are incorporated into the final object module.
> 2. Convenience - instead of having to provide many object files to the linker, we can group them unto a single static library.

## How Linkers Use Static Libraries to Resolve References
Static libraries are useful, but the way the Linux linker uses them to resolve external references is a bit confusing. 

Suppose we provide multiple archives and relolcatable object files for linking. During the symbol resolution phase, the linker scans all relocatable object files and archives in the order that they appear on the command line (left to right). The [driver](Compiler%20Drivers.md) automatically translates any `.c` files on the command line into `.o` files. During this scan, the linker maintains a set  $E$ of relocatable object files, a set $U$ of unresolved symbols, and a set $D$ of symbols that have been defined in previous input files. Initially, $E$, $U$ and $D$ are empty.

Then:

* For each input file $f$, the linker determines if $f$ is an object file or archive. If it's an object file, it adds $f$ to $E$, updates $U$ and $D$ to reflect symbol references and definitions in $f$, and proceeds.
* If $f$ is an archive, the linker attempts to match unresolved symbols in $U$ against those defined by members of the archive. If some archive member $m$ defines a symbol that resolves a reference in $U$, it adds $m$ to $E$ and updates $U$ and $D$. This process iterates over the member object files in the archive until a point where $U$ and $D$ no longer change.
* If $U$ is nonempty when the linker finishes scanning the files on the command line, it terminates with an error.

The output of this process is essentially a set $E$ of relocatable object files.

The key point here is that the ordering of libraries on the command line is significant. This can cause confusing link-time errors. For instance, if the library that defines a symbol appears on the command line before the object file that references that symbol, then the reference will not be resolved and linking will fail. 

The general rule for libraries is to place them at the end of the command line. If the members of the different libraries are independent, in that no member references a symbol defined by another member, then the libraries can be placed at the end of the command line in any order. But if the libraries are not independent, they must be ordered so that for each symbol $s$ that is referenced externally by a member of an archive, at least one definition of $s$ follows a reference to $s$ on the command line.







