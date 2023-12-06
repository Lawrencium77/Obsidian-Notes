> [!INFO]
> An **object file** is a file containing machine code output of an assembler. 

Object files come in three forms:

1. **Relocatable Object File**: Contains binary code and data in a form that can be combined with other relocatable object files at compile time, to create an executable object file.
2. **Executable Object File**: Contains binary code and data in a form that can be directly copied into memory and executed.
3. **Shared Object File**: Special type of relocatable object file that can be loaded into memory and linked dynamically, either at load or run time.

Object files are organised according to specific **object file formats**, which vary from system to system. Modern x86-64 Unix systems use the **Executable and Linkable Format (ELF)**. We will focus on ELF, but the basic concepts are similar across formats. ^34d72a

