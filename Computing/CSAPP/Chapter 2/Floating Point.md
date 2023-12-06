Since I already know a decent amount about floating point (see my [previous notes](../../Floating%20Point.md)), I didn't cover this section in all its detail. 

```toc
```

### Rounding Modes
The IEEE floating point format defines four different **rounding modes**. The default finds the closest match. The others calculate upper/lower bound.

Figure 2.37 shows these four modes:

![](_attachments/Screenshot%202023-03-14%20at%2016.59.24.png)

**Round-to-even** is so-called since it rounds all numbers to their **nearest** value, and any "ties" (i.e. multiples of $0.50) go towards the nearest even value.

We could instead round all ties upwards/downwards. But this would introduce a statistical bias. The computed average of a set of numbers would always be slightly higher/lower than the true value.

### Floating Point Operations
The IEEE standard specifies that the result of an arithmetic operation should yield:

$$\textrm{Round}(x\odot y)$$

where $x$ and $y$ are floating-point values viewed as real numbers. In other words, we should round the *exact* result of the real operation. In practice, we don't have to do the exact computation. The computation only need be sufficiently precise for the rounded result to be the same.

However, floating point addition **doesn't form an Abelian group**, since:

* It's not associative
* Inverses don't always exist, e.g. $\infty-\infty=\textrm{NaN}$

The lack of associativity is very important. It has implications for compilation. If a compiler is given the following:

```C
x = a + b + c
y = b + c + d
```

it may be tempted to save one floating point addition by doing:

```C
t = b + c
x = a + t
y = t + d
```

but this might yield a different value for `x`, since it uses a different ordering.


### Floating Point in C
I skipped this section.


