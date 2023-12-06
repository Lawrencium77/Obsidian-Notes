These notes are based on [this blog](https://lernapparat.de/jit-optimization-intro/?utm_source=pocket_mylist). In reading a bit further, I found [another blog](https://ppwwyyxx.com/blog/2022/TorchScript-Tracing-vs-Scripting/#Performance) that gives quite a nice discussion of Tracing vs Scripting in TorchScript.

I have not summarised the *entire* blog. And I have certainly not grokked its contents. But there is enough detail here to understand the narrative.
```toc
```

## Intro
**TorchScript** is the language implemented by the PyTorch JIT (*Just in Time* compiler) - PyTorch's solution for deployment and model optimisation. 
It has multiple uses:

* Export models to work beyond Python (e.g. on mobile);
* To escape the Python GIL (Global Interpreter Lock)
* Making holistic optimisations (that consider several optimisations at once)

## The Overall Structure of PyTorch

![](_attachments/Screenshot%202022-09-01%20at%2022.21.55.png)

PyTorch is most prominently a Python library - the author calls this **classic PyTorch**. Some parts are implemented in Python (e.g. `torch.nn` modules), but the compute functions (like `torch.matmul`) are provided as a Python C++ extension.

This Python C++ extension is a wrapper around PyTorch's C++ library **LibTorch**. This is turn uses the ATen tensor library which itself dispatches to various backends.

The PyTorch JIT implements a virtual machine that takes in TorchScipt programs (typically created with `torch.jit`) and runs them by calling in LibTorch itself, circumventing the PyTorch parts.

## TorchScript
The term **TorchScript** is overloaded. It refers both to:

* The language (mostly a typed subset of Python)
* The intermediate representation of the exported graph

This section focuses on how to get models into TorchScript - the form the JIT can process.

There are two main ways of achieving this (although they can be mixed): **scripting** and **tracing**.

### Scripting
Scipting compiles a subset of Python. It takes the Python source code and transforms it. We can phrase this as figuring out *"Here's what the function should do"*, just like normal programming.

For example:

![](_attachments/Screenshot%202022-09-01%20at%2022.30.42.png)

Pay attention to the output of `fn.graph`. It's clearly listing the operations, and what data they act upon.

### Tracing
Tracing runs the code and observes the calls into PyTorch with some sample input. It can be phrased something like: "Watch me, now you know how to do the same".

Another way of understanding tracing is that it runs a model with certain inputs, and "traces" (i.e. records) all the operations that are executed. It then combines these into a graph.

![](_attachments/Screenshot%202022-09-01%20at%2022.34.38.png)

### What is TorchScript?
But what actually is TorchScript?

One important difference between TorchScript and Python is that in Torchscript, **everything is typed**.

PyTorch will mostly infer the intermediate and return types, but you need to annotate any non-tensor inputs.

Another important difference is the **binding behvaiour** -  when a given variable name is looked up to find the associated variable. Python uses **late binding**: if we write a function that calls `torch.matmul`, the Python interpreter will look up what `torch.matmul` is when it executes the statement in which it is used.

This is in contrast to many other languages, which use **early (aka static) binding**. TorchScript uses this. When we compile a function to TorchScript, the JIT looks it up then and there, and puts it into our function.

### Tracing vs Scripting
So what exactly is the different between tracing and scripting?

Scripting will process all of the code, but it may not understand all of it. This means it captures all constructs it supports (e.g. some control flow), but it will fail if there is something it doesn't understand.

Tracing cannot see anything that is not a direct call into PyTorch and will happily ignore it (e.g. control flow). 

![](_attachments/Screenshot%202022-09-01%20at%2022.45.18.png)

If a model is both traceable and scriptable, **tracing generates a graph that's at least as simple as scripting**. This is because the graph compiler isn't always smart enough to optimize a model fully. For complex models, this problem is worse.

The above paragraph is based upon [this blog](https://ppwwyyxx.com/blog/2022/TorchScript-Tracing-vs-Scripting/#Performance), which argues that tracing is generally more useful than scripting.

## How the JIT works at a very high level
It will be useful to take a closer look at how the JIT works under the hood. The JIT has several phases to get us from a function to running our programs. For our purposes, we can think of the following three stages:

1. Go from tracing/source code to a graph
2. Do multiple compiler passes through the graph to go from `.graph` to an optimised graph (this can be retrieved with `.graph_for(*inputs*)`).
3. The `.graph` is compiled to a form of bytecode that is then executed by a virtual machine. This maintains the operands on a stack and then dispatches to the various operators registered by LibTorch or the *custom operators* that extend the JIT.

### Tracing or Scripting to a `.graph`
When tracing a function, the LibTorch dispatcher will call a special function for every call of a LibTorch function. Before re-dispatching to LibTorch operation, this special function records a graph node that includes function calls, source location and type information.

There is more detail on this process in the blog.

### Optimisation Passes
The JIT compiler gets us from a `.graph` to what we see with `.graph_for`. It runs a series of optimisation (and some other) passes. This is done by the JIT's GraphExecutor on the first run, or first few runs in the case of the profiling executor. The optimised graphs are cached along with the bytecode.

There are a number of passes that work and do not affect the automatic differentiation, such as:

* Elminating dead code and common subexpressions, pre-computing things that only involve constants;
* Pooling redundant constants into single values, and some simple "pattern matching" optimisations (like eliminating `.t().t()`).

### Bytecode and Execution
Finally, the optimised graph is lowered to bytecode and run by the virtual machine.

## Excursion: GPU, Efficiency, Measurement
Before we discuss optimisation through the JIT, we have to discuss measurement. How do we profile in PyTorch?

A lot of measurement can be done with very basic tools, e.g. IPython's `%timeit` magic.

GPU computations are deliberately asynchronous. It is importend to avoid unneeded synchronisation points, requiring the CPU to wait for the GPU work to finish to inspect results.

Synchronisation happens because the program needs to know something about the computation (e.g. sizes of tensors depending on the input). These are often unavoidable, but typical sources of spurious synchronisation can result from simple operators like `.to(device="cpu")`, `'.item()'`, `._to_list()` and `print`.

If we want to time GPU kernels, we want to be sure to synchronise before taking the start and end times. Typically, we also want to run some "warm-up" iterations, i.e. run the measured function a few times before timing it.

## Optimisation
### Is Python being slow an issue? (No)
The first optimisation target should be *what we compute*.

But when we have optimised what we compute, how else can we optimise?

The conventional wisdom is that Python is slow. Certainly, it isn't fast, but if the GPU is saturated and we are using asynchronous execution, then Python isn't the real bottleneck.

### How PyTorch programs spend their time
At a high level, you can divide the time spent by PyTorch programs into these parts:

* Python program flow
* Data "administrative overhead" (creating `Tensor` data structures, autograd `Nodes`, etc)
* Data aquisition (I/O)
* Computation, roughly as:
	* Fixed overhead (kernel launches etc)
	* Reading/writing memory
	* "Real computation"

As a general rule of thumb, provided your operands aren't unreasonably large, Python and data "administrative overhead" probably isn't your main problem.

So while the JIT takes away some Python overhead, this is not a spectacular optimisation. With this out of the way, let us get to how the JIT helps us optimise.

### Holistic Optimisations - JIT Fusers
Currently, the fuser is a hotspot of development. PyTorch has no fewer than **three fusers**:

![](_attachments/Screenshot%202022-09-02%20at%2015.57.54.png)

### How the JIT Optimises Pointwise Operations
To get a taste of how the JIT fuser works, let us look at the intersection over union ratio for detection models. The details aren't important here. The code looks like:

![](_attachments/Screenshot%202022-09-02%20at%2016.01.41.png)

What matters is that this is a simple function with elementwise computation. Looking at the function graph:

![](_attachments/Screenshot%202022-09-02%20at%2016.02.35.png)

It is not complex code, but it has quite a few operations. In terms of execution, every one of these ops launches a kernel (a function run on the GPU) that does three things:

* Loads the inputs from memory
* Computes the output
* Stores the result

So this function loads 37 tensors and stores 20 outputs with only trivial computation. Clearly, this is bandwidth bound.

What if we could make it all into one large kernel and have 8 loads and 1 store?

This is exactly what the fuser does, and it gives a good speedup:

![](_attachments/Screenshot%202022-09-02%20at%2016.06.21.png)

What's happening here? First, we do some type check. If that returns OK, we run a `TensorExprGroup` which will be executed as one kernel. We keep a fallback just in case. 

We will look in some detail at how these things work. But the core idea is that operations of the `TensorExprGroup` will be compiled into a single kernel that then computes the results from the input in one go.

## How the Fusers Work at a High Level
At a high level, PyTorch's fusers work in three parts:

1. In the fusion JIT compiler pass, the operations that can be fused are arranged in a **fusion group**. By looking at which operations can be fused, we get a good glimpse of what the fusers (think) they can achieve. The legacy PyTorch fuser only considers pointwise operations. The CUDA fuser (aka NVFuser) - which is conceptually close but more elaborate than the classic fuser - also handles `sum`. The TensorExpr fuser (fuser1, the default) additionally fuses pointwise and `softmax`. The newer two fusers also insert a check and an explicit fallback.
2. At some point, it compiles a kernel for the computation. Typically, this is specific to the type and shape of the inputs. For the GPU, the fuser emit CUDA C code and compile using the GPU RTF (run time compile) library. These kernels are cached.
3. When running a fusion group, the fuser needs to launch the kernel. For the newer fusers, checking whether the input matches expectations is done outside this node, but the classic fuser would do the fallback itself if needed.

One thing to know about the fallback is that it itself will be optimized by the PyTorch JIT. So when we run a function that has been optimized with fusions with incompatible parameters, the failing type check would cause the JIT to call the fallback, **which itself would then get the optimisations for these parameters** (and another level of check and fallback).

### Code Generation from TorchScript IR to GPU kernel
In addition to operator support, code generation is the second area in which each fuser has a different approach.

The CUDA fuser first transforms the TorchScript IR in the CudaFusionGroup to a Fusion IR. This is then further lowered to the kernel IR and finally translated to C++ code from which the runtime compiler generates the kernel.

The TensorExpr fuser translates the TorchScript IR into a sequence of loop-nest statements. This is the TensorExpr IR. They are then optimised and lowered before they are passed to the code generators that write kernels functions and then compile and run them.

## Automatic Differentiation in TorchScript
Things are a bit more involved when we need gradients. The default mode of the JIT is to execute the LibTorch operations and they will build an autograd graph just like in classic PyTorch. But when we want to fuse operators, this adds some complication. 

The problem is that AutoGrad needs intermediate results to compute the backward. Yet our express purpose is to skip storing and loading the intermediate results.

This is mitigated by the PyTorch JIT's own automatic differentiation mechanism, **AutoDiff** (as opposed to Autograd in non-JIT PyTorch execution).

When we run the `ratio_iou` operation with gradient-requiring inputs, we get a `DifferentiableGraph` that contains the `TensorExprGroup`:

![](_attachments/Screenshot%202022-09-03%20at%2015.50.17.png)

To understand why this is, we need to look at how AutoDiff works. It has roughly three stages:

1. The first part is a pass that creates these differentiable graphs (in the optimisations, notably before the fusing). AutoDiff has a catalogue of operations for which it can compute derivatives. It will move these into the `DifferentiableGraph`.
2. Next, when we run a graph containing `DifferentiableGraph` nodes, the second part of AutoDiff will compute the gradient by going through the nodes of the forward graph. This is a form of source-to-source differentiation. This can amend the forward to output intermediates that are then captured for the backward, similar to the `save_for_backward` mechanism in an `autograd.Function`.
3. Finally, the PyTorch AutoGrad(!) mechanism is used by making a `DifferentiableGraph` node (see diagram) that holds on to the intermediate values and, when backward is called, runs the backward graph constructed in the previous step (including letting the JIT optimize it, potentially fusing operations, etc.).

One thing to note here: the JIT currently does not have a terribly smart logic to decide which things to capture, and which to re-compute. Instead, it will mimic what AutoGrad does.

## The Profiling Executor
We mentioned that the JIT fusers will specialise on detailed tensor type information. How does it get this information? The answer is that it does so through the *Profiling Executor*, which is in charge of running the JITed graphs.

The profiling executor will record tensor type information in its profiling phase (the first few invocations). Note that it does **not** record timings.  Currently, it runs one profiling run, but this is configurable. It then uses this information to implement optimisations.

Traditionally, the same thing (obtaining tensor type information attached to every value) has been done by propagating the types from the inputs through the graph. While this works great in general, it soon hits limitations. For instance, the output shape of convolutions (and thus precise type information) depends on the value (not just the type) of the input.

So we instead observe shapes during runtime. When the JIT fuser passes (mentioned above) go to work, they find these typing annotations on all tensor values and can adjust.

**It is important not to confuse the *profiling executor* with the PyTorch *profiler* - they are two distinct things!**

Fusers 1 (NNC) and 2 (nvFuser) both require the profiling executor. In our Aladdin code, we therefore set `torch._C._jit_set_profiling_exector(True)` in `base_train.py`.









