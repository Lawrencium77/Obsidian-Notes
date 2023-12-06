The key point of this chapter is that procedure calls can incur overhead and block forms of program optimisation performed by the compiler. They re-write our [Program Example](Program%20Example.md) as:

![](_attachments/Screenshot%202023-11-10%20at%2017.24.25.png)

We avoid the call to `get_vec_element()` inside every `for` loop iteration.