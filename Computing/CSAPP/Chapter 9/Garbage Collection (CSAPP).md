With an explicit [allocator](Dynamic%20Memory%20Allocation.md) such as the C `malloc` package, an application allocates and free heap blocks with calls to `malloc` and `free`. It is the application's responsibility to free any allocated blocks it no longer needs.

Failing to free allocated blocks is a common programming error. For example, consider the following C function that allocates a block as part of its processing:

```C
void garbage()
{
	int *p = (int *)Malloc(15213);
	return; /* Array p is garbage at this point /*
}
```

Since `p` is no longer needed, it should have been freed before `garbage` returned.
Instead, the block remains allocated for the lifetime of the program, needlessly occupying heap space.

A [garbage collector](../../../Coding/Python/Garbage%20Collection.md) is a dynamic storage allocator that automatically frees allocated blocks that are no longer needed. Such blocks are called **garbage**.

In a system that supports garbage collection, applications explicitly allocate heap blocks but never free them. Instead, the garbage collector periodically identifies the garbage blocks and makes the appropriate calls to `free`.

There is an "amazing" number of approaches to garbage collection. We will limit our discussion to the [Mark&Sweep](#Mark&Sweep) algorithm, which is useful since it can be built on top of an existing `malloc` package to provide garbage collection for C and C++ programs.

## Garbage Collector Basics
* A garbage collector views memory as a directed **reachability graph**, of the form shown in Figure 9.49:

![](_attachments/Screenshot%202023-05-14%20at%2015.10.38.png)

* The nodes are partitioned into a set of **root** and **heap nodes**.
* Each heap node corresponds to an allocated block in the heap.
* A directed edge $p \to q$ means that some location in block $p$ points to some location in block $q$.
* Root nodes correspond to locations not in the heap that contain pointers into the heap. These locations can be registers, variables on the stack, or global variables in the read/write data area of virtual memory.
* We say that node $p$ is **reachable** if there exists a directed path from any root to node $p$. 
* Unreachable nodes correspond to garbage.
* A garbage collector aims to maintain some representation of the reachability graph and periodically reclaim the unreachable nodes by freeing them.
* For some languages (like C and C++), collectors cannot always maintain exact representations of the reachability graph. Such collectors are called **conservative garbage collectors**.
* They're conservative in the sense that each reachable block is correctly identified as reachable, while some unreachable nodes might incorrectly be identified as reachable.
* Collectors can provide their service on demand, or run as separate threads in parallel with the application.
* Figure 9.50 shows how we might incorporate a conservative collector for C programs into an existing `malloc` package:

![](_attachments/Screenshot%202023-05-14%20at%2015.18.19.png)

* The application calls `malloc` in the usual manner. If `malloc` can't find a free block that fits, then it calls the garbage collector. The collector identifies garbage blocks, and returns them to the heap by calling `free`.
* The key idea is that the *collector* calls `free`, instead of the application.

> [!INFO]
> I skipped a bunch of info on how Mark&Sweep garbage collectors work. These are described in sections 9.10.2 and 9.10.3.







