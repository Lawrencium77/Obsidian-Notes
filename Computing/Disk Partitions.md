A [disk](Storage%20Technologies#Disk%20Storage) [partition](https://en.wikipedia.org/wiki/Disk_partitioning) is quite simple - it's a logical isolated section of a storage device. These partitions are logical but not physical; the disk remains as a single unit but the OS treats each partition as a separate disk.

Each partition can have its own file system.

Disk partitions are typically physically contiguous on HDD. On SSD, this is less of a concern. 

They're useful for isolated different types of data. For example, one might contain the OS, and another might contain personal data.

They also allow separate OSs to coexist on the same physical disk.

A common minimal configuration for Linux systems is to use three partitions:

* One for the system files mounted at `/`
* One holding data mounted in `/home`
* A swap partition

## Partition Tables
A **partition table** is maintained on disk by the OS. It describes the partitions on that disk. 