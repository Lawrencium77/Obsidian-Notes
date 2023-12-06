**RAID** stands for **Redundant Arrays of Inexpensive Disks**.

They're essentially just a collection of multiple [disks](../../CSAPP/Chapter%206/Storage%20Technologies.md) that allow faster, bigger, and more reliable storage systems.

They use data striping to speed up read times.

They use **mirroring**, whereby we make more than one copy of each block in the system on a separate disk. This adds redundancy.

In general, there are various so-called "RAID levels", which vary in the level of redundancy and performance. See the [Wikipedia page](https://en.wikipedia.org/wiki/RAID) for a full description. The exact RAID level to use depends on what the end-user values. 



