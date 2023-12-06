Each file has a kind of low-level name - this is an [inode](4%20-%20Processes#Files).

A directory also has a low-level name - an **inode number**.
A directory contains a list of (user-readable, inode) pairs.

## Creating Files
Files can be created with the `open()` system call:

```
int fd = open("foo", O_CREAT|O_WRONLY|O_TRUNC, S_IRUSR|S_IWUSR);
```

CSAPP definitely talked about this in [Chapter 10 - System-Level IO](../../CSAPP/Chapter%2010/Chapter%2010%20-%20System-Level%20IO.md).

`open` returns a [file descriptor](4%20-%20Processes#Files). This makes sense, since file descriptors operate on a per-process basis, so all file descriptors will remain fixed throughout your program.

## Reading and Writing Files
They have quite a fun example of using `strace` to monitor the system calls made by `cat`:

![](_attachments/Screenshot%202023-11-25%20at%2009.19.42.png)

We can see the steps followed quite nicely here.

## Reading And Writing, But Not Sequentially
Sometimes, it's useful to be able to read from an offset within a file. The `lseek` syscall does this:

```
off_t lseek(int fildes, off_t offset, int whence);
```

This offset is referenced in the process's file struct. 

#### How the Kernel Manages File Structs Across Processes

Together, file structs represent all of the currently opened files in the system. 
The kernel keeps these as an array, with one lock for the entire table:

```
struct { 
	struct spinlock lock; 
	struct file file[NFILE]; 
} ftable;
```

When a file is opened, the kernel creates a `file` struct instance and returns the file descriptor to the process.

## Shared File Table Entries: `fork()` and `dup()`
A `file` struct can be shared between processes. This happens when a parent process creates a child with `fork`:

![](_attachments/Screenshot%202023-11-25%20at%2009.32.03.png)

The file struct contains a refcount for this purpose.

Another interesting case of sharing occurs with the `dup()` syscall. This allows a process to create a new file descriptor that refers to the same `file` struct as an existing file descriptor. This allows redirection of stdin/stdout.

> [!INFO]
> I skipped some chapters on more syscalls used for manipulating files and directories.

## Hard Links
When we call `rm`, it uses the `unlink()` system call. Why is this?

When we unlink a file (i.e. remove the `file` struct), the OS checks a ref count within the inode. `unlink()` decrements this refcount by one. When the refcount reaches zero, the file system frees the inode and related data blocks.

You can see the refcount of a file using `stat`. E.g:

```
❯ stat file
  File: file
  Size: 0         	Blocks: 1          IO Block: 131072 regular empty file
Device: 3ch/60d	Inode: 1672913     Links: 1
...
```

## Soft Links
[Softlinks](../../Linux/Hardlinks%20&%20Softlinks.md) do not increment the inode refcount. 

## Making and Mounting a File System
To make a file system, we use the `mkfs` tool. This takes as input some device (such as a disk partition, e.g., `/dev/sda1`) and a file system type (e.g., `ext3`). It then writes an empty file system, starting with a root directory, onto that [disk partition](../../Disk%20Partitions.md). 

Once such a file system is created, it needs to be made accessible within the file-system tree. This is achieved with the `mount` program. What this does is quite simple - it takes an existing directory as a target **mount point** and essentially pastes the directory tree at that point.

To see what's mounted on your system, you can run `mount`. 


