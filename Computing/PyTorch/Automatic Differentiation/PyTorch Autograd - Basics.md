These notes are part of a [series](Summary.md) I made about automatic differentation. They are based on [this PyTorch blog](https://pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html) and are specific to `torch.autograd`.

```toc
```

## Differentiation in Autograd

Let's take a look at how `autograd` collects gradients. We create two tensors `a` and `b` that have `requires_grad = True`. This signals to `autograd` that every operation on 
them should be tracked. We then create a tensor `Q` from `a` and `b`:

```python
import torch

a = torch.tensor([2., 3.], requires_grad=True)
b = torch.tensor([6., 4.], requires_grad=True)
Q = 3*a**3 - b**2
```

> [!INFO]
> By default, `requires_grad = False` when we call `torch.tensor()`.


The output of an operation will require gradients even if only a single input tensor has `requires_grad = True` (see [below](#^de280d)). 
This means that after creating a tensor with `requires_grad = True`, *every tensor that is dependent on* `a` will also have `requires_grad = True`. 

Suppose we wish to find the gradients of `Q` w.r.t `a` and `b`. When we call `.backward()` on `Q`, Autograd calculates these gradients and stores them in the respective tensors' `.grad` attribute. To be clear: gradients are deposited in `a.grad` and `b.grad`.

## Computational Graph
Conceptually, Autograd keeps a record of data (tensors) and all executed operations in a DAG consisting of [Function](https://pytorch.org/docs/stable/autograd.html#torch.autograd.Function) objects. In this DAG, leaves are the input tensors and roots are the output tensors. 

In a forward pass, Autograd does two things automatically:

* Run the requested operation to compute a resulting tensor;
* Maintain the operation's *gradient function* in the DAG.

This is exactly the same idea as we saw in our [simple Python implementation](A%20Simple%20Implementation.md). The backward pass kicks off when `.backward()` is called on the DAG root. Autograd then:

* Computes the gradients from each `.grad_fn`;
* Accumulates them in the respective tensor's `.grad` attribute;
* Using the chain rule, propagates all the way to leaf tensors.

Below is a visual representation of the DAG in our example:

![](_attachments/Screenshot%202022-08-29%20at%2018.08.08.png)

The arrows are in the direction of the forwards pass. The nodes represent the backward functions of each operation in the forward pass. The leaf nodes in blue represent our leaf tensors `a` and `b`.

> [!NOTE]
>  **DAGs are dynamic in PyTorch**. After each `.backward()` call, autograd starts populating a new graph. This allows the user to use control flow statements (if/and statements) in the model; you can change the shape, size and operations at every iteration if needed.

### Exclusion from the DAG
Autograd tracks operations on all tensors which have their `requires_grad` attribute set to `True`. For tensors that don't require gradients, setting `requires_grad = False` excludes it from the gradient computation DAG.

The output of an operation will require gradients even if only a single input tensor has `requires_grad = True`: ^de280d

```python
x = torch.rand(5,5)
y = torch.rand(5,5)
z = torch.rand((5,5), requires_grad=True)

a = x + y  # Will have requires_grad = False
b = x + y + z # Will have requires_grad = True
```

For more info, see these [more detailed notes](PyTorch%20Autograd%20-%20More%20Detail.md).

### The Relevance of Leaf Nodes 
Suppose a tensor has `requires_grad = True`. The fact that gradients need to be computed for a tensor does **not** mean that its `grad` attribute will be populated. This is in contrast to our [Python implementation](A%20Simple%20Implementation.md), which retained gradients for all nodes in the computational graph.

In order for its `grad` attribute to be populated, it must also be a [leaf node](https://pytorch.org/docs/stable/generated/torch.Tensor.is_leaf.html#torch.Tensor.is_leaf). 
All tensors that have `requires_grad = False` will be leaf tensors.
For tensors that have `requires_grad = True`, they will be leaf tensors if they were created by the user. This means they are not the result of an operation and so `grad_fn` is `None`.

To get `grad` populated for non-leaf tensors, we can use `retain_grad()`:

```python
a = torch.tensor([1., 2.], requires_grad=True)
b = 3*a**3
c = b * a
b.retain_grad() # Ensure b has .grad attribute populated
c.backward(gradient=torch.tensor([1.,1.]))

print(a.grad)
print(b.grad)

# Output
> tensor([12., 96.])
> tensor([1., 2.])
```










