[Linking](Educative%20Course#Section%2011%20Compiling,%20Linking,%20Makefile,%20Header%20Files) is the process of collecting various pieces of code into a single file that be be loaded into memory and executed. Linking can be performed at various stages:

* Compile Time
* Load Time (when the program is loaded into memory and executed by the **loader**)
* Run Time

Linkers play a crucial role in software development because they enable **separate compilation** - we can decompose our source code into smaller, manageable modules that can be modified and compiled separately. When we change one of these files, we simply recompile it and relink the application. 

This chapter provides a discussion of all aspects of linking. The topics that we cover are:

* [Compiler Drivers](Compiler%20Drivers.md)
* [Static Linking](Static%20Linking.md)
* [Object Files](Object%20Files.md)
* [Relocatable Object Files](Relocatable%20Object%20Files.md)
* [Symbols and Symbol Tables](Symbols%20and%20Symbol%20Tables.md)
* [Symbol Resolution](Symbol%20Resolution.md)
* [Relocation](Relocation.md)
* [Executable Object Files](Executable%20Object%20Files.md)
* [Loading Executable Object Files](Loading%20Executable%20Object%20Files.md)
* [Dynamic Linking with Shared Libraries](Dynamic%20Linking%20with%20Shared%20Libraries.md)
* [Loading and Linking Shared Libraries from Applications](Loading%20and%20Linking%20Shared%20Libraries%20from%20Applications.md)
* [Position-Independent Code](Position-Independent%20Code.md)
* [Library Interpositioning](Library%20Interpositioning.md)
* [Tools for Manipulating Object Files](Tools%20for%20Manipulating%20Object%20Files.md)
