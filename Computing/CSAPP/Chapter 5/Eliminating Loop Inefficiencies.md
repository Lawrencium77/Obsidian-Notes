The point of this section is that we can often make very simple code optimisations, by just being a little bit careful.

For instance, the procedure `combine1` as shown in [Figure 5.5](Program%20Example.md) calls function `vec_length` as the test condition of the `for` loop. This test condition can be moved outside of the loop and stored in a global variable. 

In an ideal world, this optimisation would be performed by the compiler. But [as discussed previously](Capabilities%20and%20Limitations%20of%20Optimising%20Compilers.md), they are typically very cautious about making transformations that change where or how many times a function is called. 