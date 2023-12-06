* Locality has two distinct forms: **temporal** and **spatial** locality.
* In a program with good temporal locality, an address that's referenced once will likely be referenced again in the near future.
* In a program with good spatial locality, if an address is referenced once, the program will likely reference nearby locations in the near future.
* Locality is crucial to the success of [caching](../Chapter%201/Caches%20Matter.md).

```toc
```

### Locality of References to Program Data
* Consider the following function. Does it have good locality?

```C
int sumvec(int v[N])
{
	int i, sum = 0;  
	for (i = 0; i < N, i++)
		sum += v[i];
	return sum;
}
```

* Let's look at the reference pattern for each variable:
	* `sum` is referenced once in each loop iteration $\implies$ good temporal locality
	* `sum` is a scalar $\implies$ no spatial locality 
	* Elements of `v` are read sequentially $\implies$ good spatial, poor temporal locality.
* Each variable has either good temporal or spatial locality. So `sumvec` has good locality.
* A function like `sumvec` that visits each element of a vector sequentially is said to have **stride-1 reference pattern** (with respect to element size). This is sometimes called a **sequential reference pattern**.
* Visiting every $k$-th element is called **stride-k reference pattern**. As stride increases, spatial locality decreases.
* [Stride](Ezyang%20-%20Internals%20of%20PyTorch#Strides) is also important for programs that reference multidimensional arrays. Consider this function that sums the elements of a 2D array:

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

* This reads elements in a [row-major](Ezyang%20-%20Internals%20of%20PyTorch#^d3295e) order. It enjoys good spatial locality since it references the array in the same row-major order that it is stored. The result is that we get a stride-1 reference pattern.
* Seemingly small changes can have big impact on locality. Simply swapping the `i` and `j` loops would read in a column-major order, leading to poor spatial locality. The result would be a stride-$N


### Locality of Instruction Fetches
* Since program instructions are stored in memory and must be fetched by the CPU, we can evaluate locality in this regard.
* In the first code example above, the instructions in the body of the `for` loop are executed in sequential memory order, so it has good spatial locality.[^fn1]
* Since the loop is executed multiple times, it also has good temporal locality.
* An important property of code that distinguishes it from program data is that it's rarely modified at runtime. The CPU rarely overwrites or modifies instructions.



[^fn1]: I'm not really sure what this means.