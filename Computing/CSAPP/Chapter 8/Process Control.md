Unix provides many system calls for manipulating processes from C programs. This sections describes the important functions and gives examples of how they're used. In particular, it has an interesting discussion of the `fork` function.

Nonetheless, I thought it was not noteworthy for the most part. A couple of exceptions are described below.

### Reaping Child Processes
When a process terminates for any reason, the kernel does not remove it from the system immediately. Instead, the process is kept around in a terminated state until it's **reaped** by the parent. When the parent reaps the terminated child, the kernel passes the child's exit status to the parent and then discards the terminated process.

A terminated process that has not yet been reaped is called a [zombie](4%20-%20Processes#^272c93).

> [!INFO]
> Page 789 contains details on how to write a shell program, which is kinda cool!

