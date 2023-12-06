Once the linker has completed [Symbol Resolution](Symbol%20Resolution.md), it knows the exact sizes of the code and data sections in its input object modules. It's now ready to being the **relocation** step, where it merges the input modules and assigns a run-time address to each symbol.

Relocation consists of two steps:

1. **Relocating sections and symbol definitions**: The linker merges all sections of the same type into a new aggregate section of the same type. For instance, the `.data` sections from the input modules are all merged into one. The linker then assigns new run-time memory addresses to these aggregated sections, and to each symbol *definition*. 
2. **Relocating symbol references within sections**: The linker modifies each symbol *reference* so that they point to the correct run-time addresses. 

> [!INFO]
> I didn't Anki the rest of this section. I may wish to revisit.

To perform this second step, the linker relies on data structures in the relocatable object modules called relocation entries.

## Relocation Entries
When an **assembler** generates an object module, it doesn't know where the code and data will ultimately be stored in memory. Nor does it know the locations of any externally defined symbols. So whenever the assembler encounters a reference to an object whose location is unknown, it generates a **relocation entry** that tells the linker how to modify the reference when it merges the object file into an executable. 

Figure 7.9 shows the format of an ELF relocation entry:

![](_attachments/Screenshot%202023-10-29%20at%2010.25.04.png)

The `offset` is the section offset of the reference that will need to be modified. The `symbol` identifies the symbol that the modified reference should point to. The `type` tells the linker how to modify the new reference.

ELF defines 32 different relocation types, many quite arcance. We consider only two:

1. `R_X86_64_PC32`: Relocate a reference that uses a 32-bit PC-relative address. 
2. `R_x86_64_32`: Relocate a reference that uses 32-bit absolute addressing.

## Relocating Symbol References
Figure 7.10 shows the pseudocode for the linker's relocation algorithm:

![](_attachments/Screenshot%202023-10-29%20at%2010.28.25.png)

Lines 1 and 2 iterate over each section `s` and relocation entry `r`. Assume that when the algorithm runs, the linker has already chosen run-time addresses for each section (`ADDR(s)`) and each symbol (`ADDR(r.symbol`). Line 3 computes the address in the `s` array of the 4-byte reference that needs to be relocated. If this uses PC-relative addressing, it's relocated by lines 5-9. Otherwise, it's relocated by lines 11-13. 

The best way to understand this is with an example. Let's see how the linker uses this algorithm the relocate our references in the program in [Figure 7.1](Compiler%20Drivers.md). Figure 7.11 shows the disassembled code from `main.0`, as generated by `objdump`:

![](_attachments/Screenshot%202023-10-29%20at%2010.35.49.png)

#### Relocating PC-Relative References
In line 6 in Figure 7.11, function `main` calls `sum()`, which is defined in `sum.o`. The corresponding relocation entry is show in line 7. It consists of four fields:

```
r.offset = 0xf
r.symbol = sum
r.type = R_X86_64_PC32
r.addend = -4
```

These tell the linker to modify the 32-bit PC-relative reference starting at offset `0xf` so that it'll point to the `sum` routine at runtime. Now, suppose the linker has determined that:

```
ADDR(s) = ADDR(.text) = 0x4004d0
ADDR(r.symbol) = ADDR(sum) = 0x4004e8
```

Using the algorithm in Figure 7.10, the linker first computes the run-time address of the reference (line 7):

```
refaddr = ADDR(s)  + r.offset
        = 0x4004d0 + 0xf
		= 0x4004df
```

and then updates the reference so that it'll point to the `sum` routine:

```
*refptr = (unsigned) (ADDR(r.symbol) + r.addend - refaddr)
        = (unsigned) (0x4004e8       + (-4)     - 0x4004df)
        = (unsigned) (0x5)
```

In the resulting executable file, the `call` instruction has the following relocated form:

```
4004de: e8 05 00 00 00 callq 4004e8 <sum>
```

Thus, the next instruction to execute is the first instruction of the `sum` routine, which is what we want!






