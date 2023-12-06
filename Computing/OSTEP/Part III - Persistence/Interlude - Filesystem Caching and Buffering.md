[5 - File System Implementation](5%20-%20File%20System%20Implementation.md) has some content on caching and buffering used in filesystems. I decided to split this into a separate document, since I wanted to elaborate on what they were saying.

## OSTEP - Caching and Buffering

> **The Crux: How to Reduce File System I/O Costs**
> Even the simplest operations like opening, reading, or writing a file incurs a large number of I/O operations, scattered over the disk. How can we reduce this cost?

Most filesystems aggressively used DRAM to cache important blocks. 

Early filesystems introduced a **fixed-size cache** to hold popular blocks. The replacement policy was usually LRU or some variant. This fixed-size cache would usually be allocated as 10% of total memory.

This **static partitioning** of memory, however, can be wasteful. Modern systems instead use **dynamic partitioning**. Specifically, they integrate [virtual memory](../../CSAPP/Chapter%209/Chapter%209%20-%20Virtual%20Memory.md) pages and filesystem pages into a **unified page cache**. This means that memory can be allocated more flexibly across virtual memory and the filesystem.

It's easy to see how this speeds up reading. But the effect of caching on writes is more complicated. Write traffic has to go to disk in order to be persistent, meaning caching doesn't provide such a benefit. That said, **write buffering** is used and has a number of performance benefits. First, it can **batch** some updates into a smaller set of I/Os (e.g. if an inode bitmap is updated by two different ops). Second, the system can **schedule** I/Os to improve performance. Finally, some writes are avoided altogether by delaying them. For example, if an application creates a file and then deletes it, delaying the write means it's avoided entirely. 

For these reasons, modern filesystems buffer writes in memory for anywhere between five and thirty seconds. This represents a tradeoff between performance and durability (if the system crashes before the I/O happens then the updates are lost).

Some applications (such as databases) don't enjoy this tradeoff. To avoid unexpected data loss due to write buffering, they simply force writes to disk by either:

* Calling `fsync()`
* Using **direct I/O** interfaces that work around the cache, or
* Using the **raw disk** interface and avoiding the filesystem altogether.

## An Example - Buffering in Python
The [Python docs](https://docs.python.org/3/library/functions.html#open) have some interesting references to the filesystem cache. Specifically, they mention that the `open()` function has a `buffering` argument. This allows the user to switch buffering on/off. 

Interestingly, the default buffering policy works slightly different to that described above. Instead of flushing the cache at some set time interval:

* Binary files are buffered in fixed-size chunks. In other words, the buffer will collect data of some buffer size before writing to disk.
* "Interactive" text files use line buffering. 

## More on the Unified Page Cache
In [conversation with ChatGPT](https://chat.openai.com/share/66365c26-7878-4a1e-a421-94b37046f650), I found out a few more details on exactly how this unified cache works.

First, note that the block sizes for the file data and virtual memory pages can differ. This makes sense - the size of a disk block may not equal the size of a VM page. The cache is able to handle this difference - I'm not sure how exactly.

Now, suppose I have a program that reads some data from a file and stores it in a global variable. This means I have to read a block of file data, and then ensure that a part of it exists in a VM page. What exactly happens in this case?

Clearly, the filesystem ensures that the disk block containing the file data is cached in DRAM. Here's the interesting part - to read it into the program, the OS simply memory-maps the relevant data into VM. There is a blurring of of the filesystem cache and VM pages. 

I think this is quite complicated and I doubt I'll fully understand the nuances of how this works without looking at some code. Suffice to say that in a unified cache, there is an integration of VM pages and file blocks which yields more efficient memory utilisation.

