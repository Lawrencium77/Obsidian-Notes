## Unsigned Addition
Consider two nonnegative integers $x$ and $y$, such that $0\le x,y\le2^{w}$. Their sum could require $2^{w+1}$ bits. If we maintain the sum as a $(w+1)$-bit number and add it to another value, we may require $w+2$ bits. And so on.

This continued "word size inflation" means we can't place a bound on the word size required to represent the results of arithmetic ops. Nonetheless, programming languages commonly only support fixed-size arithmetic (although Lisp supports arbitrary-size arithmetic!). 

Consider the addition of $x$ and $y$. Unsigned addition can be described as follows:

$$x+y=\begin{cases}
  x+y & x+y<2^w & \text{Normal} \\
  x+y-2^w & 2^w\le x+y <2^{w+1} & \text{Overflow}
\end{cases}$$

We can understand this equation as follows. If $x+y<2^w$, the leading bit in a $(w+1)$-bit representation is $0$. But if $2^w\le x+y <2^{w+1}$, the leading bit in the $w+1$-bit representation will be $1$. Hence discarding it is equivalent to subtracting $2^w$ from the sum.

This is an example of [modular addition](https://en.wikipedia.org/wiki/Modular_arithmetic). Modular addition also forms an **abelian group**. In other words, it's commutative, associative, has an identity, and every element has an additive inverse.

## Two's-Complement Addition

Two's-Complement Addition is defined as follows:

$$x+y=\begin{cases}
  x+y-2^w & x+y\ge2^{w-1} & \text{Positive Overflow} \\
  x+y & 2^{w-1}\le x+y <2^{w-1} & \text{Normal} \\
  x+y+2^w & x+y <-2^{w-1} & \text{Negative Overflow} 
\end{cases}$$

Two's-complement addition has the exact same bit-level representation as unsigned addition. In fact, most computers use the same machine instruction to perform either unsigned or signed addition.

This can only be the case since two's-complement addition is equivalent to converting the arguments to unsigned ints, performing unsigned addition, and then converting back to two's-complement. The proof for this is given in the textbook (p127).

## Unsigned Multiplication

Take integers $x$ and $y$, represented as $w$-bit unsigned numbers. Their product can range between $0$ and $(2^w-1)^2=2^{2w}-2^{w+1}+1$. This could require as many as $2w$ bits to represent.
Instead, unsigned multiplication in C is defined as follows:

$$x *y=(x\cdot y)\mod2^w$$

This follows since we are truncating the output value to $w$ bits.

## Two's-Complement Multiplication
Again, we truncate the output of multiplication to $w$ bits. As with addition, truncated two's-complement multiplication is equivalent to first computing its value modulo $2^w$ and then converting from unsigned to two's complement:

$$x*y=\textrm{U2T}((x\cdot y)\mod2^w)$$
Again, this means that the bit-level representation of the product operation is identical for both unsigned and two's-complement multiplication.







