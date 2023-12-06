I was inspired to make these notes when reading [5 - File System Implementation](OSTEP/Part%20III%20-%20Persistence/5%20-%20File%20System%20Implementation.md).

A [Virtual Filesystem](https://en.wikipedia.org/wiki/Virtual_file_system) is an abstract layer on top of a more concrete filesystem. Its purpose is to allow applications to access different types concrete filesystems in a uniform way.

For example, Linux provides a VFS. Under the hood it could be simultaneously using ext3, ext4, NFS, etc. But the user would be unaware of this.

