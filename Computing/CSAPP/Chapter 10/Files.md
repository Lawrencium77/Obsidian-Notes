Each Linux file has a **type** that indicates its role in the system:

* **Regular files** contain arbitrary data. Applications often distinguish between **text files**, which contain only ASCII characters, and **binary files**, which contains everything else. To the kernel there is no difference between text and binary files.
* **Directories** contain and array of **links**, where each link maps a **filename** to a file. Each directory contains at least two entries: `.` and `..`. 
* **Sockets** are files that are used to communicate with another process across a network.

Other files types include named [pipes](5%20-%20Interprocess%20Communication%20Mechanisms#Pipes) and [symbolic links](../../Linux/Hardlinks%20&%20Softlinks.md), but are beyond the scope of [this chapter](Chapter%2010%20-%20System-Level%20IO.md).

The Linux kernel organises all files into a single directory hierarchy, anchored by the root directory named `/`. Figure 10.1 shows a portion of this directory hierarchy:

![](_attachments/Screenshot%202023-06-03%20at%2020.04.01.png)

As part of its [context](4%20-%20Processes#Process%20Context), each process has a **current working directory** that identifies its current location in the directory hierarchy. The `cd` command changes the shell's working directory.