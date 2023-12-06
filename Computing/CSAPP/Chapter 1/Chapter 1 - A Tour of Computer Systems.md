
These notes cover chapter 1 of CSAPP.

## Contents
1. [Information is Bits + Context](Information%20is%20Bits%20+%20Context.md)
2. [Programs are Translated by Other Programs into Different Forms](Programs%20are%20Translated%20by%20Other%20Programs%20into%20Different%20Forms.md)
3. [It Pays to Understand How Compilation Systems Work](It%20Pays%20to%20Understand%20How%20Compilation%20Systems%20Work.md)
4. [Processors Read and Interpret Instructions Stored in Memory](Processors%20Read%20and%20Interpret%20Instructions%20Stored%20in%20Memory.md)
5. [Caches Matter](Caches%20Matter.md)
6. [Storage Devices form a Hierarchy](Storage%20Devices%20form%20a%20Hierarchy.md)
7. [The Operating System Manages the Hardware](The%20Operating%20System%20Manages%20the%20Hardware.md)
8. [Systems Communicate with Other Systems Using Networks](Systems%20Communicate%20with%20Other%20Systems%20Using%20Networks.md)
9. [Important Themes](Important%20Themes.md)

Firstly, a definition: 

> [!INFO]
> An **application program** (aka **application**, or **app**) is any program designed to carry out a specific task other than one relating to the operation of the computer itself. Their outputs are typically for end users.

We begin by tracing the lifetime of a `hello` program. This chapter will introduce the key concepts, terminology and components that come into play. Later chapters expand on these ideas.

```C
# include <stdio.h>

int main()
{
	printf("hello, world\n")
	return 0
}
```








