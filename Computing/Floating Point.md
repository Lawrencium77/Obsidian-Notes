These are a few notes on the **precision** of floating point numbers.

```toc
```

### Regions of Precision
The formula for FP32 is:

$$(-1)^s\times1.f \times2^{e-127}$$

This means that the precision of floating point values can be split into multiple **regions**. For $e=0$, we have a precision of $2^{-127+23}=2^{-150}$. For $e=1$, the precision is $2^{-149}$. And so on.

This idea can be seen if we look carefully at the figures on the [Wikipedia page](https://en.wikipedia.org/wiki/Floating-point_arithmetic). It's also illustrated here:

![](_attachments/Pasted%20image%2020230209190945.png)

** **

**Question:** What are the **smallest** two consecutive values in FP32 that have a difference > 1?

**Answer:**
In FP32 $s, f, e$ posses 1, 23, 8 bits respectively.

The smallest (relative) amount by which we can increment the significand is therefore $2^{-23}$. So for a given exponent $e$, our precision is $2^{-23}\times2^{e-127}=2^{e-150}$.

Hence for $e=150$, our precision is $1$. For $e=151$, our precision is $2$. Therefore FP32 can represent

$$\dots,2^{24}-2,2^{24}-1, 2^{24}, 2^{24}+2, 2^{24}+4,\dots$$

hence the smallest two consecutive values that are representable and have a difference $>1$ are $(2^{24}, {24}+2)$.  

** **
### Subnormal Numbers
[Subnormal numbers](https://en.wikipedia.org/wiki/Subnormal_number) are a subset of floating point numbers. They're designed to fill the underflow gap around zero.

Consider FP32. When the exponent is $00_\textrm{H}$ or $FF_\textrm{H}$ the floating point number is interpreted specially:

![](_attachments/Screenshot%202023-02-11%20at%2015.06.16.png)

When $e=0, f \neq 0$ we enter the **subnormal regime** where the defining equation becomes:

$$(-1)^s\times0.f \times2^{-126}$$

This means that our FP numbers can represent far smaller numbers than would otherwise be possible:

![](_attachments/Screenshot%202023-02-11%20at%2014.53.39.png)
("denormalized" is another term for subnormal).

The minimum positive normal value is $2^{-126} \approx 1.18 \times 10^{-38}$.
The minimum positive subnormal value is $2^{-149}\approx 1.4 \times 10^{-45}$ .

** **
Subnormal numbers result in a kind of **asymmetry** in floating point. 
In **FP16**, the maximum representable value is $(2-2^{10})\times2^{15}$. 
The minimum positive subnormal value is $2^{-24}$.

This results in the following kind of behaviour:

```python
N = 2 ** 16 
print(1 / (torch.tensor(N).half())) # Gives 0
print((1 / torch.tensor(N)).half()) # Gives 1.5259e-05
```

In the first case, $2^{16}$ is too large for FP16 to represent, so calling `.half()` results in an overflow. But calling `.half()` on $2^{-16}$ is fine.




