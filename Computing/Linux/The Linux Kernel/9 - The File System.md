This chapter describes how the Linux kernel maintains the files in the [filesystems](../../OSTEP/Part%20III%20-%20Persistence/5%20-%20File%20System%20Implementation.md) that it supports.  I thought it a little esoteric but made some notes on a few interesting bits.
## Registering File Systems
The `/proc/filesystems` file shows all of the filesystems that are supported by the kernel, either by being directly built into the kernel or as LKMs.


## The `/proc` File System
The `/proc` filesystem doesn't really exist. So how can you `cat /proc/devices`? 
When the VFS makes calls to the `/proc` filesystem requesting inodes as its files are opened, the `/proc` filesystem creates those files and directories from information within the kernel. 

## Device Special Files
Linux presents its hardware devices as special files. For example, `/dev/null` is the null device. A device file doesn't use any space in the file system; it is only an access point to the device driver. The Linux VFS implements device files as a special type of inode. 

When an I/O request is made to a device file, it is forwarded to the appropriate device driver within the system. There are two types of device file; **character** and **block** special files:

* Character special files provide unbuffered, direct access to the hardware device. They handle data one character at a time, hence their name. They;'re used for devices that need to be read/written character by character, e.g. keyboards, mice, etc.
* Block special files provide [buffered](../../OSTEP/Part%20III%20-%20Persistence/Interlude%20-%20Filesystem%20Caching%20and%20Buffering.md) access to the device. They handle data in blocks. They're typically used for secondary storage.






