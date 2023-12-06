TL;DR: Gradient accumulation helps imitate a larger batch size in instances where GPU memory is a limiting factor.

**Motivation:** 

* As DL models increase in size, it becomes more difficult to fit such networks on GPU memory. This is especially relevant in situations where our data requires a lot of memory, such as computer vision
* We may be forced to use smaller batches during training --> slower convergence and lower accuracy.

**Solution:**

* We can increase the effective batch size by using **gradient accumulation**. 
* Simply speaking, this means we use a small batch size but save the gradients. We then update the network weights once every few batches.

A schematic of this process is shown below:

![](_attachments/Screenshot%202022-02-25%20at%2016.19.48.png)