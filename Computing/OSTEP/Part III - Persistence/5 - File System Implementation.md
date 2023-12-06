> [!INFO]
> I went into a bit more detail on this chapter. I think it was important.

The file system is pure software. A simple definition that I saw is:

> [!QUOTE]
> A Filesystem is just a layer in order to get some meaning to the bytes [stored on disk].

In general, the word "filesystem" can be quite ambiguous. It is [used to mean several different things](https://superuser.com/questions/292748/is-file-system-part-of-operating-system):

1. The entire system of files, rooted at `/`. I think this is basically described the **Virtual Filesystem**. With this meaning, people talk of POSIX operating systems having a "single _filesystem tree_".
2. Another word for a [volume](../../Volumes.md).
3. A component of the OS that presents a tree of files to the rest of the system. This interpretation is the focus of this chapter.

Many different filesystems have been built over the years; we will be learning about a simple file system (**vsfs**) in this chapter in order to introduce the key concepts.

> THE CRUX: HOW TO IMPLEMENT A SIMPLE FILE SYSTEM
> How can we build a simple file system? What structures are needed on disk? What do they need to track? How are they accessed?
> 

# The Way to Think
The authors suggest thinking about two different aspects of filesystems. If you understand both, you probably understand how the filesystem basically works:

* The **data structures** of the filesystem - what type of on-disk structures does it use?
* Its **access methods** - how does it map calls made by a process, such as `open()`, `read()`, `write()`, etc. onto its structures? 

Try to work on developing your mental model throughout this chapter.

# Overall Organisation
We now consider the on-disk organisation of the data structures of the vsfs filesystem. 

Our view of the disk partition where we're building the filesystem is simple: a series of $N$ blocks, of each size 4 KB. Assume we have a really small disk, with just 64 blocks:

![](_attachments/Screenshot%202023-11-26%20at%2016.49.14.png)

Let's think about what we need to store in these blocks. The first thing that comes to mind is user data. We call this region of the disk the **data region** and, for simplicity, reserve a fixed portion of 56 blocks for this:

![](_attachments/Screenshot%202023-11-26%20at%2016.59.50.png)

The filesystem also has to track metadata about each file. It does so in a structure called an [inode](4%20-%20Files%20and%20Directories.md). To accommodate these, we reserve some space on disk for them. We call this array of inodes the **inode table**. Let's assume we use 5 of our 64 blocks for inodes:

![](_attachments/Screenshot%202023-11-26%20at%2017.01.21.png)

Inodes are typically not that big, for example 128 or 256 bytes. Assuming 256 bytes per inode, a 4KB block can hold 16 inodes. Our filesystem can therefore hold $5 \times 16 = 80$ inodes. This represents the maximum number of files we can have in our filesystem. 

We also need some way to track whether inodes or data blocks are free or allocated. Such **allocation structures** are a required component of any file system. One possible allocation structure is an [Implicit Free List](Dynamic%20Memory%20Allocation#Implicit%20Free%20Lists). But we choose a structure called a **bitmap**. We use one for the data region and one for the inode region.

A bitmap is a simple structure - each bit indicates whether the corresponding object/block is free or in-use. Our new on-disk layout is:

![](_attachments/Screenshot%202023-11-28%20at%2009.40.33.png)

where $i$ corresponds to the inode bitmap, and $d$ is the data bitmap. It is overkill to use an entire 4-KB block for these bitmaps. But we do this for simplicity.

Finally, we reserve the last block for the **superblock**, denoted $S$ below:

![](_attachments/Screenshot%202023-11-28%20at%2009.41.50.png)

The superblock contains info about the particular filesystem including, for example, how many inodes and data blocks it possesses, where the inode table begins, and so on. When mounting the filesystem, the OS will read the superblock first to initialise various parameters, and then attach the volume to the filesystem tree.

## The Inode
Each inode is implicitly referred to by a number (called the **i-number**). In vsfs, given an i-number, you should directly be able to calculate where on the disk the corresponding inode is located. For instance, we have the following layout for the beginning of the vsfs filesystem partition:

![](_attachments/Screenshot%202023-11-28%20at%2009.45.48.png)

To read inode number 32, the filesystem would first calculate the offset into the inode region ($32 \cdot \textrm{sizeof(inode)}=8192$), add it to the start address of the inode table on disk (12 KB), and thus obtain the correct start address of 20 KB. 
Recall that disks are not byte addressable but rather consist of a series of blocks. To fetch the block of inodes that contains inode 32, the filesystem would issue a read to sector $\frac{20 \times 1024}{512} = 40$.

Inside each inode is virtually all the information you need about a file: its **type**, **size**, **protection information**, etc. In addition, one of the important decisions on the design of an inode is how it refers to where data blocks are. One simple approach would be to have one or more **direct pointers** (disk addresses) inside the inode; each pointer refers to one disk block that belongs to the file. Such an approach is limited. For instance, if you want to have a file that is really big (e.g., bigger than the block size multiplied by the number of direct pointers in the inode), you are out of luck.

### The Multi-Level Index
To support bigger files, one common idea is to have a special pointer known as an **indirect pointer**. Instead of pointing to a block that contains user data, this points to a block that contains more pointers, each of which points to user data. This allows us to accommodate for large files by allocating indirect blocks when needed. 

Not surprisingly, this approach can be extended to use **double-indirect pointers** and **triple-indirect pointers**. 

This overall *imbalanced-tree* approach is called a **multi-level index** approach to pointing to file blocks. Many filesystems use this, including commonly-used ones such as Linux `ext2` and `ext3`. 

You may wonder: why use imbalanced trees like this? Why not a different approach? It turns out that as researchers have studied filesystems, they've found a set of certain "truths" that holds across time. One such finding is that **most files are small**. This imbalanced tree design reflects this reality - if most files are small, it makes sense to optimise for this case.

## Directory Organisation
In vsfs (and many other filesystems), directories have a simple organisation; they're just a list of [entry name, inode number] pairs. 
For example, assume a directory `dir` has three files in it: `foo`, `bar`, and `foobar_is_a_pretty_longname`, with inode numbers 12, 13, and 24. The on-disk data for dir might look like:

![](_attachments/Screenshot%202023-11-28%20at%2010.01.23.png)

`strlen` records the length of the name, and `reclen` records the total number of bytes for the name plus any leftover space.

Deleting a file can leave empty space in the middle of a directory. We therefore need a way to mark this, e.g. with a reserved inode number such as zero.

Of course, this simple linear list of directory entries is not the only way to store information. But it is the simplest.

## Free Space Management
Free space management means tracking inodes and data blocks that are free, and those that are not. Vsfs uses two bitmaps for this task.

When we create a file, we'll allocate an inode for that file. The filesystem thus searches through the bitmap for an inode that is free, and allocates it to the file. The filesystem will have to mark the inode as used and update the bitmap accordingly. A similar set of activities takes place when a data block is allocated.

# Access Paths: Reading and Writing
We are now in a position to understand the flow of operation that happens when reading or writing a file. Understanding this **access path** the second key to understanding how filesystems work - pay attention!

For the following examples, we assume the filesystem has been mounted and thus the superblock is already in memory. Everything else (inodes, directories) is still on disk.

## Reading a File from Disk
Let's assume you want to open a file `/foo/bar`, read it, and then close it. We'll assume the file is 12 KB in size (3 blocks).

When you issue an `open("/foo/bar", O_RDONLY)` call, the filesystem must **traverse** the pathname and thus locate the desired inode.

All traversals being at the root directory. The inode for the root directory is "well-known"; the FS must know what it is when the filesystem is mounted. 

Once the inode is read in, the FS can look inside to find pointers to data blocks, which contain the contents of the root directory. The FS will use these on-disk pointers to search for an entry for `foo`, and thus its inode number.

This happens recursively until the desired inode is found. The final step of `open()` is to read `bar`'s inode into memory. It then returns a file descriptor to the user.

Once open, the program can issue a `read()` call. The first read will read in the first block of the file, consulting the inode to find it. 

At some point, the file will be closed. There is much less work to be done here; the file descriptor is deallocated, and that's it.

A depiction of this entire process is shown here:


![](_attachments/Screenshot%202023-11-28%20at%2019.00.35.png)

## Writing a File to Disk
Writing to a file is a similar process. First, it must be opened. Then, the application calls `write()` to update the file with new contents. Finally, the file is closed.

Unlike reading, writing may also **allocate** a block. When writing out a new file, each write not only has to write data to disk but also first decide which block to allocate to the file and thus update other structures accordingly (e.g. data bitmaps and inode). 
Overall, each write to a file logically generates five I/Os:

* One to read the data bitmap
* One to write the bitmap
* Two more to read & write the inode
* One to write the actual block

File creation is even more complicated. The system must not only allocate an inode, but also allocate space within the directory containing the new file. 

## Caching and Buffering
See [Interlude - Filesystem Caching and Buffering](Interlude%20-%20Filesystem%20Caching%20and%20Buffering.md).

## Interaction with Device Drivers

When a file is read or written, the filesystem issues commands to the [device driver](../../Device%20Driver%20Types.md). The driver, in turn, communicates these commands to the hardware device.