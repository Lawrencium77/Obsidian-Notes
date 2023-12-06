Figure 7.3 shows the format of a typical [ELF](Object%20Files#^34d72a) relocatable object file:

![](_attachments/Screenshot%202023-10-26%20at%2019.38.41.png)

The **ELF header** contains information about the word size and byte ordering of the system that generated the file. It then contains info that allows a linker to parse and interpret the object file. This includes the size of the ELF header, the object file type (e.g., relocatable, executable, or shared), the machine type (x86-64), and the file offset of the section header table. The section header table described the locations and sizes of the various other sections.

Sandwiched between the ELF header and the section header table are the following sections: ^d8c6e7

* `.text` - The machine code of the compiled program
* `.rodata` - Read-only data such as [jump tables](Control#^585231) .
* `.data` - **Initialised** global and static C variables (local variables are maintained on the stack).
* `.bss` - **Uninitialised** global and static C variables, along with any others that are initialised to zero. This section occupies no actual space in the object file; it's just a placeholder. At run time, these variables are allocated in memory with an initial value of zero.
* `.symtab` - A **symbol table** with information about functions and global variables that are defined and referenced in the program. Some programmers mistakenly think that a program must be compiled with `-g` to get symbol table information. In fact, every relocatable object file has a symbol table in `.symtab` (unless the programmer has specifically removed it with the `strip` command). Unlike the symbol table inside a compiler, the `.symtab` table doesn't contain entries for local variables.
* `.rel.text` - A list of locations in the `.text` section that will need to be modified when the linker combines this object file with others. In general, any instruction that calls an external function or references a global variable will need to be modified. Note that relocation info isn't needed in executable object files, and is usually omitted.
* `.rel.data` - Relocation information for any global variables that are referenced or defined by the module. In general, any initialised global variable whose initial value is the address of a global variable or externally defined function will need to be modified.
* `.debug` - A debugging symbol table with entries for local variables and typedefs, and the original C source file. This is only present if `gcc` is called with `-g`.
* `.line` - A mapping between source code line numbers, and machine code instructions. Again, this is only present if `-g` is used.
* `.strtab` - A string table for the symbol tables in the `.symtab` and `.debug` sections. A string table is a sequence of null-terminated character strings. ^c19352

