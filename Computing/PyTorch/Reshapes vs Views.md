These are just a few notes that provide extra detail on [reshapes](https://pytorch.org/docs/stable/generated/torch.reshape.html) and [views](https://pytorch.org/docs/stable/generated/torch.Tensor.view.html) in PyTorch.

#### Views sometimes fail with noncontiguous inputs
`torch.view()` raises a `RuntimerError` when the input is noncontiguous:

```python
import torch

x = torch.randn(6,2)
y = x.t()
y.is_contiguous() # False
y.view(6,2)
```

```
RuntimeError: view size is not compatible with input tensor's size and stride (at least one dimension spans across two contiguous subspaces). Use .reshape(...) instead.
```

The reason for this is that if the input is not contiguous its behaviour will be unpredictable. For example:

```python
import torch

x = torch.arange(1,7)
y = x.reshape(3,2)[:,0] # tensor([1, 3, 5])
y.data_ptr() == x.data_ptr() # True
```

Graphically, this corresponds to:

![](_attachments/Screenshot%202023-01-30%20at%2011.25.42.png)

(The array represents the storage of `x`).

Taking a view of `y` would be problematic since we could end using elements from its storage that don't belong to the actual tensor. 

#### Reshape returns a view if and only if the input is contiguous
`torch.reshape()` will **always** return a view if the input is contiguous.
But if the input is non-contiguous, it must first create a copy.

