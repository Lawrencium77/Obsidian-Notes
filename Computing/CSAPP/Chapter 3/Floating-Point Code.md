> [!INFO]
> The key idea here is that x86-64 maintains a separate set of registers to handle floating-point data.

To understand the x86-64 floating-point architecture, it's helpful to have a historical perspective.

Since 1997, both Intel and AMD have incorporated successive generations of ***media*** instructions to support graphic and image processing. These instructions originally focused on [SIMD](Important%20Themes#^7a80f9) parallelism. The most recent version of these instructions is [AVX](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions). AVX manages a set of "YMM" registers that are 256 bits in size.

Since 2000, these media instructions have included ones to operate on ***scalar*** floating point data, using single values in the low-order 32/64 bits of YMM registers. 

We focus on AVX2, the second version of AVX which was introduced in 2013. As illustrated in Figure 3.45, AVX2 allows data to be stored in 16 YMM registers, each of which is 256 bits long:

![](_attachments/Screenshot%202023-10-26%20at%2015.59.05.png)

The general style of machine code generated for operating on floating-point data with AVX2 is similar to what we've seen previously with integer data. Both use a collection of registers to hold and operate on values, and they use these registers for passing function arguments. 

Due to this similarity, I decided to not go into the rest of the chapter in any detail.