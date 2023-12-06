Vectorization is a really important part of making tensor-processing code run fast. But how does it actually work?

As a reminder, vectorization refers to executing operations on entire arrays rather than individual elements of data:

```python
import numpy as np

# Without vectorization
list1 = [1, 2, 3, 4, 5]
list2 = [6, 7, 8, 9, 10]
sum_list = []

for i in range(len(list1)):
    sum_list.append(list1[i] + list2[i])
print(sum_list) 

# With vectorization
arr1 = np.array([1, 2, 3, 4, 5])
arr2 = np.array([6, 7, 8, 9, 10])
sum_arr = arr1 + arr2
print(sum_arr)  
```

Its implementation is rooted in the hardware architecture of modern CPUs, in particular their ability to perform [SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data) operations. CPU instruction sets include special SIMD operations. These operate on small fixed-size arrays, also called vectors, that are loaded into special registers in the CPU. For example, with 128-bit SIMD instructions, one could load four FP32 values into a single SIMD register and perform the same operation on all four numbers, in a single clock cycle.

**Numpy** can exploit this since it's built on top of lower level libraries, such as C and Fortran, which have access to these low-level instructions. C/C++ does provide functions that map directly to SIMD instructions. Modern compilers can also do "auto-vectorization" to automatically convert some parts of your code into SIMD instructions. The ability of the compiler to do this is highly dependent on the specific code, the compiler itself, and the compiler's settings.

Of course, vectorization is not only a feature of CPUs. [GPUs](GPUs/GPUs.md) use similar concepts, but on a larger scale.




