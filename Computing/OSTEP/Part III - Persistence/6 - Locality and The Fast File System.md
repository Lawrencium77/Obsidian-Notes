An inode should be put close to its data blocks.

We inevitably end up with external [fragmentation](Dynamic%20Memory%20Allocation#Fragmentation) too, e.g.:

![](_attachments/Screenshot%202023-12-03%20at%2011.55.34.png)

where `A1` is the first block of file `A`, etc.

Disk **defragmentation** tools help with this; they reorganise on-disk data to make files as contiguous as possible.

There is also a trade-off with block size: smaller blocks yield less internal fragmentation, but are bad when reading larger files as there is more repositioning overhead when moving the disk head between blocks.

## Disk Awareness
> Crux: How to make the file system "disk aware"?

The **Fast Filesystem (FFS)** divides the disk into a number of **cylinder groups**. Each cylinder is a set of tracks on different HDD surfaces that are the same distance from the centre. The key idea is to make the filesystem **disk aware**.

> [!INFO]
> I read the rest of the subchapter but didn't make any notes on it.