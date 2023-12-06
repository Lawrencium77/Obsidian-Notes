Modern compilers employ sophisticated algorithms to determine what values are computed in a program and how they're used. They can then exploit opportunities to simplify expressions, reduce the number of times a given computation is performed, etc.

Compilers must be careful to apply only **safe** optimisations to a program, meaning they cannot change program behaviour. This can be more restrictive than one might assume. For instance, consider the following two procedures:

```C
void twiddle1(long *xp, long *yp)
{
	*xp += *yp
	*xp += *yp
}

void twiddle2(long *xp, long *yp)
{
	*xp += 2* *yp
}
```

In most cases, both procedures have identical behaviour but `twiddle2` is more efficient. Hence, if a compiler is given `twiddle1` to compile, we might think it could generate more efficient code based on `twiddle2`.
However, consider the case where `xp = yp`. Then `twiddle1` will do:

```C
*xp += *xp; /* Double value at xp */
*xp += *xp; /* Double value at xp */
```

such that it quadruples the value at `xp`, whereas `twiddle2` triple it:

```C
*xp += 2* *xp /* Triple value at xp */
```

The case where two pointers designate the same memory location is called **memory aliasing**. To perform safe optimisations, the compiler must assume that different pointers may be aliased. This is an example of an **optimisation blocker**: an aspect of a program that can limit opportunities for optimisation.

Another example is function calls. For instance:

```C
long counter = 0;

long f() {
	return counter++;
}

long func1() {
	return f() + f() + f() + f();
}

long func2() {
	return 4*f();
}
```

It might seem at first that both `func1` and `func2` compute the same result. But in practice, `f()` has a side effect of modifying some global state. So changing the number of times it gets called changes the program behaviour.

Most compilers do not try to determine whether a function is free of side effects, and hence is a candidate for optimisations such as those attempted in `func2`.

## Optimising Functions Calls by Inline Substitution
Code involving function calls can be optimised by **inlining**, where the function call is replaced by the code for the body of that function. For example, `func1` can be expanded as:

```C
long func1in() {
	long t = counter++;
	t += counter++;
	t += counter++;
	t += counter++;
	return t;
}
```

This transformation helps in two ways. First, it removes the overhead of the function calls. Second, it allows further optimisation of the expanded code, e.g:

```C
long func1opt() {
	 long t = 4 * counter + 6;
	 counter += 4;
	 return t
}
```




