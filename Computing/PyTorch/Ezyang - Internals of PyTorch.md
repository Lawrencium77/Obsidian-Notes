These notes are based upon [a blog post](http://blog.ezyang.com/2019/05/pytorch-internals/) by ezyang on the internals of PyTorch. The intended audience is developers who have used PyTorch, but who haven't necessarily delved into how a machine learning library is written.

[These slides](https://fleuret.org/dlc/materials/dlc-handout-1-6-tensor-internals.pdf?utm_source=pocket_mylist) cover similar stuff, but in a little less detail.

The blog is split into two parts. The first covers the concepts involved in a tensor library. The second grapples with the actual details of coding in PyTorch. In these notes, I focus on the former.
```toc
```

## Part I - Concepts
### The Tensor
The **tensor** is the central data structure in PyTorch. We all have an intuitive idea of what it represents: it's an $n$-dimensional data structure containing of some sort of type (floats, ints, etc).
We can think of a tensor as consisting of data, and then some metadata describing the size of the tensor, the type of element it contains (dtype), and what device the tensor lives on (CPU? CUDA?).

![](_attachments/Screenshot%202022-09-29%20at%2011.06.28.png)

#### Strides
There is another piece of metadata you might be less familiar with: the **stride**. Strides are a distinctive feature of PyTorch, so they're worth discussing.

![](_attachments/Screenshot%202022-09-29%20at%2011.09.28.png)

A tensor is a mathematical concept. But to represent them on computers, we have to define a physical representation for them. The most common is to lay out each element contiguously, writing each row to memory. This is shown above. In this example, we have specified that the tensor contains 32-bit integers, so we can see that each integer lives at a physical address, each offset 4 bytes from the other. 

So what do strides have to do with this picture?

![](_attachments/Screenshot%202022-09-29%20at%2011.11.48.png)

Suppose we want to access the element at position [0,1] in our logical representation. How do we translate this logical position to a location in physical memory? Strides tell us how to do this: to find out where any element for a tensor lives, we multiply each index with the respective stride for that dimension, and sum them all together.

In the above picture, the first dimension is colour coded blue, and the second is coded red. Doing the sum, we get two (zero-indexed), and indeed, the number three lives two below the beginning of the contiguous array.

Strides are the fundamental basis of how we provide views to PyTorch users. For instance, suppose we want to extract a tensor that represents the second row of the tensor above:

![](_attachments/Screenshot%202022-09-29%20at%2011.16.25.png)

In PyTorch, we can just write `tensor[1,:]` to get this row. Here's the important thing: when we do this, we don't create a new tensor; instead, we return a tensor which is a different *view* on the underlying data.
This means that if we edit the data in that view, it will be reflected in the original tensor. See my notes on [Reshapes vs Views](Reshapes%20vs%20Views.md) for more info on views.

This example was simple because three and four live in contiguous memory, so all we need to do is record an offset saying that the data of this (logical) tensor lives two down from the top. (Every tensor records an offset, but most of the time it's zero, and we'll omit it from diagrams when that's the case).

A more interesting case is if we want to take the first column:

![](_attachments/Screenshot%202022-09-29%20at%2011.18.59.png)

When we look at physical memory, we see that the elements of the column are not contiguous: there's a gap of one element between them. Here, strides come to the rescue: instead of specifying a stride of one, we specify a stride of two. This says that between one element and the next, you need to jump two slots.

#### Side Note: What Exactly is a Contiguous Tensor?
> [!INFO]
> This section is not in the original blog. I added it myself.

This section adds some detail to the idea of a *contiguous* tensor.

The term "contiguous" is actually a little confusing. A discontiguous tensor does **not** have its contents spread out in disconnected blocks of memory. Instead, it means that bytes are allocated in the same block of memory, but the ordering of how to access this data is somewhat different to the default.

Specifically, a tensor whose values are laid out contiguously in memory when reading in a **row-major** order (i.e. rightmost direction first) are contiguous. This is convenient because we can visit them efficiently without jumping around the storage. ^d3295e

As an example, consider this tensor (of shape `(3,2,2)` and stride `(4,2,1)`):

![](_attachments/Screenshot%202022-10-03%20at%2020.00.52.png)

We can take a view of this storage to create a tensor of shape `(2,2,3)` and stride `(6,3,1)`:

![](_attachments/Screenshot%202022-10-03%20at%2019.59.55.png)

We can express contiguity mathematically:

> [!INFO]
> A tensor of shape $(a,b,c)$ is contiguous **if and only if** it has strides $(bc,c,1)$.
> This generalises to tensors with any number of dimensions.

### Tensors & Storage

Let's take a step back for a moment, and think about how we would actually implement this functionality. If we can have views on a tensor, this means we have to decouple the notion of the **tensor** (the user-visible concept), and the actual physical data that stores the data of the tensor (called **storage**).

![](_attachments/Screenshot%202022-10-03%20at%2019.40.42.png)

There may be multiple tensors which share the same storage. Storage defines the `dtype` and physical size of the tensor. The tensor defines the sizes, strides and offset - which define the logical interpretation of the physical memory.

It's important to realise that there is a always a pair of Tensor-Storage, even for "simple" cases where you don't really need storage (e.g. you just allocated a contiguous tensor with `torch.zeros(2,2)`).

> [!INFO]
> PyTorch are interested in making this Tensor-Storage picture *not* true; instead of having a separate concept of storage, just define a view to be a tensor that is backed by a base tensor. This is a little more complex, but it has the benefit that contiguous tensors get a much more direct representation (as the Storage indirection is avoided). A change like this would make PyTorch's internal representation more like Numpy's
#### Viewing a Tensor's Storage in PyTorch
> [!INFO]
> This is another section that I added myself.

I thought it would be handy to make a note of a few functions that can be used to view a tensor's storage in PyTorch.

Let's begin with a tensor `x`:

![](_attachments/Screenshot%202022-10-04%20at%2014.39.13.png)

We can examine its storage with `x.storage()`:

![](_attachments/Screenshot%202022-10-04%20at%2014.39.39.png)

We see that this shows us a 1D version of the tensor data. The storage object also has an associated dtype and size. These can be shown with `x.storage().size()` and `x.storage.dtype`. We can also view the offset of a Tensor in the underlying storage with `x.storage_offset()`:

![](_attachments/Screenshot%202022-10-04%20at%2014.42.24.png)

We can now see that creating a view of `x` creates a new Tensor with the same underlying storage. In this case, `y` is a view of `x`:

![](_attachments/Screenshot%202022-10-04%20at%2014.43.10.png)

So the Storage is the same:

![](_attachments/Screenshot%202022-10-04%20at%2014.43.32.png)

### Tensor Extensions
There is more to life that dense, CPU float tensors. There is a range of extensions, such as XLA tensors, quantized tensors, or MKL-DNN tensors.

![](_attachments/Screenshot%202022-10-04%20at%2009.10.00.png)

The current PyTorch model for extensions offers four extension points on tensors. First, there is the trinity three parameters which uniquely determine what a tensor is:

* **Device**: The description of where the tensor's physical memory is actually stored, e.g. on a CPU, on an Nvidia GPU (cuda), or perhaps an AMD GPU (hip) or a TPU (xla). The distinguishing feature of a device is that it has its own allocator, that doesn't work with any other device.
* **Layout**: Describes how we logically interpret this physical memory. The most common layout is a strided tensor. But sparse tensors have a different layout involving a pair of tensors - one for indices, and one for data.
* **dtype**: describes what is actually stored in each element of the tensor. This could be floats or integers. Or it could be, for example, quantized integers.

The Cartesian product of these parameters define all of the possible tensors you can make. Not all of these combinations may actually have kernels (who's got kernels for sparse, quantized tensors on an FPGA?) but *in principle* the combination could make sense.

There's one last way you can make an "extension" to Tensor functionality, and that's writing a wrapper class around PyTorch tensors that implements your object type. But I'm not gonna worry too much about this!

### Autograd
If tensors were the only thing that PyTorch provided, it'd basically just by a Numpy clone. The distinguishing characteristic of PyTorch when it was originally released was that it provided automatic differentiation on tensors. These days, there are other cool features like TorchScript; but back then, that was it!

What does automatic differentiation do? It's the machinery that's responsible for taking a neural network:

![](_attachments/Screenshot%202022-10-04%20at%2009.18.36.png)

and filling in the missing code that actually computes the gradients of the network:

![](_attachments/Screenshot%202022-10-04%20at%2009.19.09.png)

Take a moment to study this diagram. There's a lot to unpack; here's what to look at:

1. First, look at the variables in red and blue. PyTorch implements **reverse-mode automatic differentiation**, which means we effectively walk the forwards computation "backward" to compute the gradients. You can see this if you look at the variable names: at the bottom of the red, we compute `loss`; then, the first thing we do in the blue part of the program is compute `grad_loss`. Technically, these variables which we call `grad_` are not really gradients; they're really Jacobians left-multiplied by a vector, but in PyTorch we just call them `grad` and mostly everyone knows what we mean.
2. If the structure of the code stays the same, the behaviour doesn't. Each line from forwards is replaced with a different computation, that represents the derivative of the forward operation. For example, the `tanh` operation is translated into a `tanh_backward` operation.

The whole point of autograd is to do the computation that is described by this diagram, but without actually ever generating this source. PyTorch autograd doesn't do a source-to-source transformation.

To do this, we need to store metadata when we carry out operations on tensors. Let's adjust our picture of the tensor data structure: now instead of just a tensor which points to storage, we have a *variable* which wraps this tensor, and also stores more information (called **AutogradMeta**), which is needed for performing autograd when a user calls `loss.backward()` in their PyTorch script.

![](_attachments/Screenshot%202022-10-04%20at%2009.33.16.png)

For more detail on autograd, see my notes on [Summary](Automatic%20Differentiation/Summary.md).

> [!WARNING]
> This diagram may be out of date. There was an [MR](https://github.com/pytorch/pytorch/issues/13638) to merge variables and tensors in PyTorch.


