This set of notes is based upon [this](https://horace.io/brrr_intro.html) blog post.
```toc
 style: bullet
 min_depth: number (default: 2)
 max_depth: number (default: 6)
 title: Table of Contents
 allow_inconsistent_headings: boolean (default: false)
 delimiter: string (default: |)
 varied_style: boolean (default: false)
```

## Intro

Our motivation here is that, whilst DL systems can feel as much like alchemy as they do science, reasoning from first principles can still eliminate broad swathes of approaches and improve our intuition.

We can understand the efficiency of our DL regime as having 3 components:

1. Compute: Time spent on your GPU actually doing FLOPS.
2. Memory: Time spent transferring tenors within a GPU.
3. Overhead: Everything else.

Just as for training ML models, knowing what regime you're in allows you to narrow in on optimizations that matter.

There is an idea here that behind [The Bitter Lesson](../../ML/The%20Bitter%20Lesson.md) is a legion of engineers that keep [GPUs](GPUs.md) running efficiently.

## Compute
One perspective on DL is that we'd like to maximize the time spent in the compute-bound regime. 

This can be difficult. Available compute grows faster than [memory bandwidth](#^6af533):

![](_attachments/Screenshot%202022-03-17%20at%2015.32.52.png)

One way to think about compute is as a factory. We send instructions (overhead) and materials (memory-bandwidth) to keep our factory running efficiently (compute).

![](_attachments/Screenshot%202022-03-17%20at%2015.34.54.png)

If our factory increases processing capacity faster than the rate at which we can supply materials, it becomes harder for our factory to achieve peak efficiency:

![](_attachments/Screenshot%202022-03-17%20at%2015.35.47.png)

There is one useful point to note about FLOPS. Modern ML accelerators all have hardware specialized for matrix multiplication, e.g. NVIDIA's A100s:

![](_attachments/Screenshot%202022-03-17%20at%2015.39.00.png)

This means that, if you're not doing matmul, you'll only be able to achieve 19.5 TFLOPS instead of the state 312. This isn't unique to GPUs. In fact, TPUs are even *less* general than GPUs.

The fact that GPUs are so much slower at everything that isn't matmul might seem problematic. What about other operations, like layer norm or activation functions? The truth is that these other operators are just rounding errors in terms of FLOPs. For example, look at this table of FLOP counts on [BERT](../../ML/Transformers/BERT.md) for different operator types, where "Tensor Contraction" means matmul:

![](_attachments/Screenshot%202022-03-17%20at%2015.42.20.png)

Altogether, our non-matmul ops make up only 0.2% of FLOPs. It shouldn't matter than our GPU computes non-matmul ops 15x slower.

However, in this case, the normalisation and element-wise ops actually achieve **250x less and 700x less FLOPs respectively**.

Why do these non-matmul ops take so much more time than they should?

The culprit is to do with memory bandwidth...

## Bandwidth
Bandwidth costs are essentially the cost paid to move data from one place to another. This might involve moving data from: ^926433
* CPU to GPU ("data transfer costs");
* one node to another ("network costs");
* CUDA global memory to CUDA shared memory ("bandwidth cost"). ^d3f6af

In particular, the last one is what we'll be focusing on here.

To understand what the memory bandwidth cost is, let's think about our factory analogy.

Although our factory is where we do the actual work, it's not suitable as a bulk storage unit. A large part of this is that, since we're doing actual work here all the storage is optimized for being fast to actually *use* (SRAM).

Our results and materials are intead stored in a warehouse - probably somewhere cheaper and where there's a lot of space (DRAM). We then move supplies to and from our factories:

![](_attachments/Screenshot%202022-03-17%20at%2015.59.10.png)

The cost of moving stuff to and from our compute units is what's called the **memory bandwidth cost**. ^6af533

As an aside, the GPU's DRAM is what shows up in `nvidia-smi` and is the primary quantity responsible for *CUDA Out of Memory* errors.

One thing to note is that every single time we perform a GPU kernel, we need to move data from and back to our GPU's DRAM.

Now, imagine what happens when we perform a [unary](https://en.wikipedia.org/wiki/Unary_operation) operation like `torch.cos`. We need to move data from storage to the factory, then perform a tiny bit of computation for each piece of data, and then ship that storage back. Nearly all of our time is spent shipping data around, and *not* on the actual computation itself.

This is an example of a **memory-bound operation**. It's not ideal, so what 
can we do about it? 

### Operator Fusion

Let's look at how a sequence of operators might look:

![](_attachments/Screenshot%202022-03-17%20at%2016.32.03.png)

Clearly, it makes more sense to keep the data in local memory, perform all of our compute, and then send it back:

![](_attachments/Screenshot%202022-03-17%20at%2016.32.44.png)

This is **operator fusion** - the most important optimisation in DL compilers. One example is [Fused Multiply Add](GPUs.md#^77feae). Simply put, instead of writing our data to global memory just to read it again, we avoid several memory accesses by performing several computations at once.

For example, if we do `x.cos().cos()` this would require 4 global reads and writes:

```python
x1 = x.cos() # Read from x in global memory, write to x1
x2 = x1.cos() # Read from x1 in global memory, write to x2
```

With operator fusion, we only need 2 global memory reads and writes:

```python
x2 = x.cos().cos() # Read from x in global memory, write to x2
```
There are a couple of issues that make this a little tricky. First, the GPU needs to know what's going to happen when performing the current operation. So you can't do this optimization in eager-mode, where PyTorch runs operators one at a time. Second, we need to generate CUDA code for this, which is a whole other thing!

If you're interested in writing custom CUDA kernels, this is likely where you'll see the most benefit. Any 2 PyTorch operators present an opportunity for fusion. If you want to try writing some custom CUDA kernels, [Triton](https://openai.com/blog/triton/) is a great place to start. In addition, many existing compilers can often perform "simple" fusions - NVFuser and XLA being two examples.

Finally, operator fusion leads to some surprising consequences. For one, a fused `x.cos().cos()` will take nearly the exact same time as calling `x.cos()` by itself. This is why activation functions are nearly all the same cost.

This fact leads to some interesting consequences for rematerialisation/activation checkpointing. Essentially, doing extra recomputation might lead to _less_ memory-bandwidth, and thus less runtime. Thus, we can lower both memory _and_ runtime through rematerialisation!

### Reasoning about Memory-Bandwidth Costs
When figuring out if an operation is memory-bandwidth bound, a calculator can go a long way.

For simple operators, we can reason about memory bandwidth directly. For example, an A100 has 1.5 TB/s of global memory bandwidth, and can perform 19.5 TFLOP/s of compute. So, if you're using 32 bit floats (i.e. 4 bytes), you can load in 400 billion numbers in the same time that the GPU can perform 20 trillion operations. Moreover, to perform a simple unary operator (like multiplying a tensor by 2), we actually need to *write* the tensor back to global memory.

So, unless you're doing about 100 FLOPs in your unary operator, you'll be spending more time performing memory accesses than actual compute.

For larger systems, it's often more difficult to say whether you're compute bound or memory-bandwidth bound, since they contain a mixture of both components.

One common approach is to measure your achieved FLOPs as a percentage of peak FLOPS. For example, if you're achieving 80% of your peak FLOPS, then you know that you're at least 80% compute bound, which is pretty good! The rest of your time is probably spent doing memory-bandwidth operations.

However, in addition to memory-bandwidth costs, there's one more thing that might cause you GPUs to underperform...

## Overhead

Overhead is when your code is spending time doing anything that's **not** transferring tensors or doing things. For example, time spent in the Python interpreter, PyTorch framework, and launching (but not executing) CUDA kernels.

The primary reason that overhead is such a pernicious problem is that modern GPUs are *really* fast. In comparison, Python is *really slow*. An A100 can perform 312 **trillion** FLOP/s, whereas Python can do ~30 million additions per second.

That means that in the time that Python can perform a _single_ FLOP, an A100 could have done through **9.75 million FLOPS**. Even worse, the Python interpreter isn't even the only source of overhead - frameworks like PyTorch also have many layers of dispatch before you get to your actual kernel.

Given this, you might be shocked that anybody uses PyTorch at all, but keep in mind that modern deep learning models are often performing **massive** operations. Moreover, frameworks like PyTorch execute _asynchronously_. That is, while PyTorch is running a CUDA kernel, it can continue and queue up more CUDA kernels behind it. So, as long as PyTorch can "run ahead" of the CUDA kernels, most of the framework overhead gets completely hidden!

![](_attachments/Screenshot%202022-03-17%20at%2017.11.57.png)

So, how do you tell if you're in this regime? Well, since overhead generally doesn't scale with problem size (while compute and memory do), the easiest way to tell is to simply increase the size of your data. If that doesn't increase the runtime proportionally, you're overhead bound. For example, if you double your batch size but your runtime only increases by 10%, you're likely overhead bound.

Another way is to use the PyTorch profiler. Here, the pink lines actually show how the CPU kernels match up with the GPU kernels.

![](_attachments/Screenshot%202022-03-17%20at%2017.12.52.png)

## Conclusion
If you want to speed up your deep learning system, the most important thing is to understand what the bottleneck in your model is.

![](_attachments/Screenshot%202022-03-17%20at%2017.13.56.png)