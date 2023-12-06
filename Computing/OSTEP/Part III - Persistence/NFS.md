I made notes on the first page of the [NFS chapter](https://pages.cs.wisc.edu/~remzi/OSTEP/dist-nfs.pdf). I thought it'd be useful for a little context.

The basic setup of an [client-server](../../../Speechmatics/Pipeline%20Efficiency/Our%20Client-Server%20Model.md) filesystem is:

![](_attachments/Screenshot%202023-12-03%20at%2012.13.52.png)

This setup allows for easy sharing of data between clients. 

Figure 49.2 shows the basic architecture of a distributed filesystem:

![](_attachments/Screenshot%202023-12-03%20at%2012.15.05.png)

On the client-side, there are client applications which access files by issuing syscalls to the client-side filesystem. To the client application, the filesystem doesn't appear to be any different from a local filesystem (except perhaps for performance).

Both the client and server-side operating systems contain a networking layer, which allows the client-side filesystem to communicate with the server-side filesystem. 