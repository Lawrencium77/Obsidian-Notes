I wasn't 100% clear on what [broadcasting](https://pytorch.org/docs/stable/notes/broadcasting.html) is. So I made some notes. It's worth noting that broadcasting is part of NumPy, as described [here](https://numpy.org/doc/stable/user/basics.broadcasting.html).

```toc
```

## What is Broadcasting?
A **broadcast** operation refers to the automatic resizing of tensors to perform operations between tensors of different shapes.

When you perform an operation between tensors with different shapes, PyTorch automatically broadcasts the smaller tensor to have the same shape. This is done by replicating the elements in the smaller tensor to match the shape of the larger tensor.

Here's an example:

```python
import torch

a = torch.tensor([[1], [2], [3]]) # (3, 1)
b = torch.tensor([[4, 5, 6, 7]])  # (1, 4)

# a and b are broadcast to (3, 4)
c = a + b

print(c)
# Output: tensor([[ 5,  6,  7,  8],
#                 [ 6,  7,  8,  9],
#                 [ 7,  8,  9, 10]])

```

## What are its benefits?
The main benefit is **memory consumption**: we can automatically expand tensors to be of equal size (without making copies of the data).

## Broadcasting Rules
This section is based on the [PyTorch docs](https://pytorch.org/docs/stable/notes/broadcasting.html).

### When Can we Broadcast?
Two tensors are broadcastable if:

1. Each tensor has at least 1 dimension
2. When iterating over dimension sizes, starting at the trailing dimension, the dimension sizes are equal, one of them is 1, or one of them doesn't exist.

For example:

```python
# broadcastable, because x does not have at least 1 dimension
x=torch.empty((0,))
y=torch.empty(2,2)

# can line up trailing dimensions
x=torch.empty(5,3,4,1)
y=torch.empty(  3,1,1)

# not broadcastable, since in the 3rd trailing dimension 2 != 3
x=torch.empty(5,2,4,1)
y=torch.empty(  3,1,1)
```

### Shape of Output
If two tensors `x` and `y` are broadcastable, then the resulting tensor size is calculated as follows:

* If the ranks of `x` and `y` are not equal, prepend 1 to the dimensions of the tensor with fewer dimensions to make them equal.
* Then, for each dimension size, the resulting dimension size is the max of the sizes of `x` and `y` along that dimension.






