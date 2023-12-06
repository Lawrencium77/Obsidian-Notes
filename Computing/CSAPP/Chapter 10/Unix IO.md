A Linux [file](Files.md) is a sequence of $m$ bytes:

$$B_0, B_1, \dots, B_{m-1}$$
Here's a key point:

> [!IMPORTANT]
> All I/O devices, such as networks, disks, and terminals, are modelled as files. All I/O is performed by reading and writing these files. This is sometimes summarised as "[everything is a file](https://en.wikipedia.org/wiki/Everything_is_a_file)".

The advantage of this is that all I/O operations can be performed in a consistent way. These include opening, closing, reading, and writing files.

