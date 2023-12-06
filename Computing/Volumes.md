These notes describe [volumes](https://en.wikipedia.org/wiki/Volume_(computing)#:~:text=In%20computer%20data%20storage%2C%20a,partition%20of%20a%20hard%20disk.), and how they differ to [Disk Partitions](Disk%20Partitions.md).

A volume is a logical section of physical storage that's managed by a single file system. They can therefore be mounted and accessed separately. In Linux, volumes are mounted at different points in the directory structure.

But it's not the same thing as a partition. A partition does not necessarily have a volume associated with it. After a disk partition is created, it is formatted with some file system. This formatted partition is called a volume.

There's often, but not always, a one-to-one mapping between disk partitions and volumes. E.g. in some [RAID](OSTEP/Part%20III%20-%20Persistence/3%20-%20RAIDs.md) configurations, multiple partitions can be combined to form a single logical volume. 

> [!SUMMARY]
> Quite a nice summary is that we *partition* a disk, *format* a partition, and *mount* a volume.




