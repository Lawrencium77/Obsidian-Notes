There are a number of Linux tools to help understand object files. These include:

* `ar` - Creates static libraries, and inserts, deletes, lists, and extracts members.
* `strings` - Lists all printable strings in an object file.
* `strip` - Deletes symbol table information from an object file.
* `nm` - Lists the symbols defined in the symbol table of an object file.
* `size` - Lists the names and sizes of the sections of an object file.
* `readelf` - Displays the complete structure of an object files. Subsumes the functionality of `size` and `nm`.
* `objdump` - The mother of all binary tools. Can display all of the information in an object file. Its most useful function is as a disassembler. 

Linux systems also provide the `ldd` tool for manipulating shared libraries:

* `ldd` - Lists the shared libraries that an executable needs at run time.

