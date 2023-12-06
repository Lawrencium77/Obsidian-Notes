> [!INFO]
> I didn't Anki this section. I may wish to revisit.

We have seen how the linker merges multiple object files into a single executable file. Figure 7.13 summarises the information in a typical ELF executable file:

![](_attachments/Screenshot%202023-10-29%20at%2010.44.18.png)

The format of an executable object file is similar to that of a [relocatable object file](Relocatable%20Object%20Files.md). In particular:

* The ELF header describes the overall format of the file. It also includes the program's **entry point**, which is the address of the first instruction to execute when the program runs.
* The `.text`, `.rodata`, and `.data` sections are similar to those in a relocatable object file, except that they have been relocated to their eventual run-time addresses. 
* The `.init` section defines a small function, `_init()`, that is called by the program's initialisation code. 

ELF executables are designed to be loaded into memory, with contiguous chunks of the executable file mapped to contiguous memory segments. This mapping is described by the **segment (aka program) header table**. Figure 7.14 shows part of this table for our example executable `prog`:

![](_attachments/Screenshot%202023-10-29%20at%2010.48.30.png)

We see that two memory segments will be initialised with the contents of the executable object file. Lines 1 and 2 tell use that the first segment (the **code segment**) has read/execute permissions, starts at memory address `0x400000`, has a total size of `0x69c` bytes, and is initialised with the first `0x69c` bytes of the executable object file. 

Lines 3 and 4 tell us that the second segment (the **data segment**) has read/write permissions, starts at memory address `0x600df8`, has a total size of `0x230` bytes, and is initialised with `0x228` bytes in the `.data` section starting at offset `0xdf8` in the object file. 

For any segment, $s$, the linker must choose a starting address that is aligned as specified in the program header ($2^{21}=$`0x200000`). I think the reason for this is that it aligns the program sections along page boundaries, so is more efficient when transferring data between main memory and disk. ^0ef570



