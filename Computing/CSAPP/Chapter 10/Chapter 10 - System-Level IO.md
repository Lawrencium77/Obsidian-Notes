**Input/Output (I/O)** is the process of copying data between main memory and external devices, such as [disk](Storage%20Technologies#Disk%20Storage). An input operation copies data from an I/O device to main memory, and vice versa.

All language run-time systems provide high-level facilities for performing I/O. For instance, C provides the **standard I/O** library. On Linux systems, these high-level functions are implemented using system-level [Unix I/O](Unix%20IO.md) functions, provided by the kernel. 

This chapter introduces the general concepts of Unix I/O and standard I/O. Its contents are:

* [Unix IO](Unix%20IO.md)
* [Files](Files.md)
* Opening and Closing Files
* Reading and Writing Files
* Robust Reading and Writing with the RIO Package
* Reading File Metadata
* Reading Directory Contents
* [Sharing Files](Sharing%20Files.md)
* [IO Redirection](IO%20Redirection.md)
* Standard IO

I only made notes on some of these subchapters.