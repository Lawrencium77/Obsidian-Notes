To begin, a definition: in the context of linking, a **symbol** is a named identifier that represents a specific piece of code/data. Symbols can be function names, variable names, constant names, etc. 
During compilation, the compiler translates high-level constructs into low-level machine instructions. During this process, it might produce an IR where symbols are still present. This IR is often stored in object files.

Each relocatable object module, $m$, has a symbol table. In the context of a linker, there are three different kinds of symbols:

* **Global Symbols** that are defined by $m$ and that can be referenced by other modules. Global linker symbols correspond to **non-static** C functions and global variables.
* Global symbols that are referenced by $m$ but defined elsewhere. Such symbols are called **externals**. They correspond to non-static C functions and global variables that are defined in other modules.
* **Local Symbols** that are defined and referenced exclusively by $m$. These correspond to **static** C functions and global variables (that are defined by the `static` attribute). These symbols are visible anywhere within $m$, but cannot be referenced elsewhere.

> [!INFO]
> #### Side Note: The `static` keyword in C
> In C, the `static`  keyword can be applied to a variable in two scenarios:
> 1. If a local variable within a function is declared as `static`, it retains its value between function calls.
> 2. When you declare a global variable or function as static, its scope is limited to the file where it's defined. Other source files in the same project can't directly access that variable or function. 

It's important to realise that local linker symbols do not describe local variables. In other words, the symbol table in `.symtab` doesn't contain any symbols that correspond to local non-static program variables. These are instead managed at runtime, on the stack.

However, local procedure variables that are defined with the C `static` attribute are not managed on the stack. Instead, the compiler allocates space in `.data` or `.bss` for each definition and creates a local linker symbol in the symbol table with a unique name. 

Symbol tables are built by assemblers, using symbols exported by the compiler. An ELF symbol table contains an array of entries, the format of each entry being as follows:

![](_attachments/Screenshot%202023-10-26%20at%2020.03.38.png)

The `name` is a byte offset into the [string table](Relocatable%20Object%20Files#^c19352) that points to the null-terminated string name of the symbol. The `value` is the symbol's address. For relocatable modules, the `value` is an offset from the beginning of the section where the object is defined. For executable object files, it's an absolute run-time address. The `size` is the size (in bytes) of the object. The `type` is usually either `data` or `function`. The symbol table can also contain entries for the individual sections and for the path name of the original source file. 

Each symbol is assigned to some section of the object file, denoted by the `section` field, which is an index into the [section header table](Relocatable%20Object%20Files#^d8c6e7). There are three special pseudosections that don't have entries in the section header table:

* `ABS` - symbols that should not be relocated. 
* `UNDEF` - undefined symbols, i.e. those that are referenced in this object module but are defined elsewhere.
* `COMMON` - uninitialised data objects that are not yet allocated.

Note that these pseudosections only exist in relocatable object files; they don't exist in executable object files.

The distinction between `COMMON` and `.bss` is subtle. Symbols are assigned using the following convention:

![](_attachments/Screenshot%202023-10-26%20at%2020.11.44.png)

The reason for this stems from the way the linker performs symbol resolution, which is explained in [Section 7.6](Symbol%20Resolution.md). 

The GNU `readelf` program is a handy tool for viewing the contents of object files.  For example, here are the last three symbol table entries for the relocatable object file `main.o`, from the example in Figure 7.1:

![](_attachments/Screenshot%202023-10-26%20at%2020.16.54.png)

We see an entry for the definition of global symbol `main`, a 24-byte function located at an offset of 0 in the `.text` section. This is followed by the definition of a global symbol `array`, an 8-byte object located at an offset of zero in `.data`. The last entry comes from the reference to the external symbol `sum`. `readelf` identifies each section by an integer index. `Ndx=1` indicates `.text`, `Ndx=3` indicates `.data`, etc.

