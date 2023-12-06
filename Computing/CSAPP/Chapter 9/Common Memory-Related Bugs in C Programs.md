This section is certainly useful, and shouldn't be ignored. However, I think it lends itself to learning by *doing* rather than through a textbook, so I only made notes on the important parts.

## Dereferencing Bad Pointers
There are larges holes in a process's virtual address space that are not [mapped](Memory%20Mapping.md) to any meaningful data. If we attempt to [dereference](Educative%20Course.md#Dereferencing%20a%20Pointer) a pointer into one of these holes, the OS will terminate our program with a [segfault](VM%20as%20a%20Tool%20for%20Memory%20Protection.md). 

## Reading Uninitialised Memory 
Heap memory is **not always initialised to zeros**. This is despite it being [demand-zero](Memory%20Mapping#^b05890); it's possible for a block to have been previously used and then `free`d, so it may contain leftover data.

This is a common error that programmers make, for instance:

```C
/* Return y = Ax  */
int *matvec (int **A, int *x, int n)
{
	int i, j;

	int *y = (int *)Malloc(n * sizeof(int));

	for (i = 0; i < n; j++)
		for (j = 0; j < n; j++)
			y[i] += A[i][j] * x[j];
	return y;
}
```

Here, the programmer has incorrectly assumed that `y` is initialised to zero. A correct implementation would explicitly zero `y[i]`.

## Memory Leaks
[Memory leaks](https://en.wikipedia.org/wiki/Memory_leak) occur when programmers fail to free allocated blocks. For instance:

```C
void leak(int n)
{
	int *x = (int *)Malloc(n * sizeof(int));
	return; /* x is garbage at this point */
}
```

If `leak` is called frequently, then the heap will gradually fill up with garbage. 



