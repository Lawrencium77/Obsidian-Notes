* Now that we understand how [Cache Memories](Cache%20Memories.md) work, we can be more precise about how locality improves code runtime.
* Programs with better locality have lower [miss rates](Cache%20Memories#^240296).
* We should write code that's **cache friendly**, in the sense that it has good locality.
* The basic approach we use is:

> 1. **Make the common case go fast**. Programs spend most of their time in a few core functions. These functions spend most of their time in a few loops. So focus on the inner loops and core functions, and ignore the rest.
> 2. **Minimise the number of cache misses in each inner loop**. Loops with better miss rates run faster.

* Consider the `sumvec` function from the section on [Locality](Locality.md):

```C
int sumvec(int v[N])
{  
	int i, sum = 0; 
	for (i = 0; i < N; i++)
		sum += v[i];
    return sum;
}
```

* Is this function cache friendly?
* First, notice that the inner loop has good temporal locality w.r.t `i` and `sum`.
* Since these are local variables, any reasonable compiler will cache them in the register file.
* Now consider the stride-1 references to vector `v`. In general, if a cache has block size of $B$ bytes, then for stride-$k$ reference pattern (where $k$ is expressed in words) the average misses per loop iteration is:

$$\min(1, (\textrm{word size}\times k)/B)$$
* This is minimised for $k=1$.
* Suppose that `v` is block-aligned, words are 4 bytes, cache blocks are 4 words, and the cache is initially empty. Regardless of the cache organisation, the references to `v` will give the following pattern of hits and misses:

![](_attachments/Screenshot%202023-04-04%20at%2020.06.35.png)

* In this example, the reference to `v[0]` misses and the corresponding block, which contains `v[0]` - `v[3]`, is loaded into the cache from memory. So the next three references are all hits.
* The same is true for `v[4]`-`v[8]`.
* In general, 3/4 references will hit. Which is the best we can do in this case with a cold cache.
* To summarise, the simple `sumvec` example illustrates two important points:

> 1. Repeated references to local variables are good because the compiler can cache them in the register file (temporal locality)
> 2. Stride-1 reference patterns are good because caches at all levels of the memory hierarchy store data as contiguous blocks.

* Spatial locality is especially important in programs that operate on multi-dimensional arrays. Consider the `sumarrayrows` function, that we also saw in the section on [Locality](Locality.md):

```C
int sumarrayrows(int a[M][N])
{
	int i, j, sum = 0;
	for (i = 0; i < M; i++)
		for (j = 0; j < N; j++)
			sum += a[i][j]
	return sum;
}
```

* C stores arrays in row-major order. So the inner loop has the same stride-1 reference pattern as `sumvec`.
* Making the same assumptions about the cache as for `sumvec`, the references to `a` will result in the following hits & misses:

![](_attachments/Screenshot%202023-04-04%20at%2020.12.01.png)




