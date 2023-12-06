## Integral Data Types
**Integral** data types are those that represent finite ranges of integers. This figure shows those available in 32-bit C programs:

![](_attachments/Screenshot%202023-03-11%20at%2019.10.44.png)

Note that the ranges are **not symmetric**: the range of negative numbers extends further than positive numbers.

## Unsigned Encodings
Consider an integer of $w$ bits. The definition of an unsigned encoding is, for vector $\boldsymbol{x}=[x_{w-1},x_{w-2},\dots,x_0]$:

$$B2U_w(\boldsymbol{x})=\sum_{i=0}^{w-1}x_i2^i$$

The function $B2U$ means "Binary to Unsigned". This feels like a fancy way of explaining binary numbers!

The unsigned binary encoding has the important property that every number between $0$ and $2^{w-1}$ has a unique encoding. In other words, $B2U$ is a **bijection**.

## Two's Complement Encodings
How to represent negative numbers too? For this, we use **two's-complement** form. This is defined by interpreting the most significant bit to have negative weight:

$$B2T_w(\boldsymbol{x})=-x_{w-1}2^{w-1}+\sum_{i=0}^{w-2}x_i2^i$$
The most significant bit is also called a **sign bit**. This formulation yields the positive-negative asymmetry described above:

![](_attachments/Screenshot%202023-03-11%20at%2019.22.05.png)

Let's consider the range of values that can be represented by a $w$-bit two's complement number.
The most negative representable value is $[10\cdots0]=-2^{w-1}$ 
The most positive representable value is $[01\cdots1]=2^{w-1}-1$

Two's complement encodings are also unique. Again, $B2T$ is a bijection.

It's also worth noting that $0$ and $-1$ have the same representations in all word sizes:

$$0=0\textrm{x}00 \newline, -1=0\textrm{xFF} $$

> [!INFO]
> There is a bunch more detail in the textbook about how to convert from signed to unsigned, truncate, expand, etc. I didn't think it super important.




