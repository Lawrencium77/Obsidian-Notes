These notes are part of a [series](Summary.md) I made about automatic differentation. They are based on [this PyTorch blog](https://pytorch.org/docs/stable/notes/autograd.html#:~:text=Autograd%20is%20reverse%20automatic%20differentiation,roots%20are%20the%20output%20tensors.) and are specific to `torch.autograd`.

```toc
```

### How Autograd Encodes the History
Conceptually, Autograd records a graph of all the operations that created the data as you execute operations. This gives a DAG whose leaves are the input tensors and roots are the output tensors. 
By tracing this graph from root to leaves, you can automatically compute gradients using the chain rule.
See [these slightly more basic notes](PyTorch%20Autograd%20-%20Basics.md) for another description.

Internally, Autograd represents this graph as a graph of [Function](https://pytorch.org/docs/stable/autograd.html#torch.autograd.Function) objects which can be `apply()`ed to compute the result of evaluating the graph. When computing the forwards pass, autograd simultaneously performs the requested computations and builds up a graph representing the function that computes the gradient (the `.grad_fn` attribute of each `torch.Tensor` is an entry point into this graph).

It's important to note that the graph is created from scratch at each iteration - it is built **dynamically**. One benefit is that this allows for control flow in the model. 

### Saved Tensors
Some operations need intermediate results to be saved during the forward pass in order to execute the backward pass. For instance, the function 

$$f(x) = x^2$$
saves the input $x$ to compute the gradient.

When defining a custom Autograd function, you can use the `save_for_backward()` function to save tensors during the forward pass. You can then use `saved_tensors` to retrieve them during the backward pass.
For example:

```python
import torch

class LegendrePolynomial(torch.autograd.Function):
    @staticmethod
    def forward(ctx, input):
        """
		ctx is a context object that can be used
        to stash information for backward computation.
        """
        ctx.save_for_backward(input)
        return 0.5 * (5 * input ** 3 - 3 * input)

    @staticmethod
    def backward(ctx, grad_output):
        input, = ctx.saved_tensors
        return grad_output * 1.5 * (5 * input ** 2 - 1)
```

For operations that PyTorch defines, tensors are automatically saved as needed.
You can explore which tensors are saved by looking for attributes of a tensor's `grad_fn` with prefix `_saved`:

```python
x = torch.randn(5, requires_grad=True)
y = x.pow(2)
print(x.equal(y.grad_fn._saved_self))  # True
```

### Locally Disabling Gradient Computation
There are several mechanisms available to locally disable gradient computation.

To do so across entire blocks of code, there are context managers like `no_grad()` and `inference_mode()`. For more fine-grained exclusion of subgraphs from gradient computation, there is setting the `requires_grad` field of a tensor.

> [!WARNING]
> There also exists eval mode (`nn.Module.eval()`). This is not actually used to disable computation but is often mixed up with the other methods

#### Setting `requires_grad`
`requires_grad` defaults to `False` unless wrapped in an `nn.Parameter`. 

During the forward pass, an operation is only recorded in the backward graph if at least one of its input tensors has `requires_grad = True`. During the backward pass, only leaf tensors with `requires_grad = True` will have gradients accumulated in their `.grad` fields.

Setting `requires_grad = False` should be the main way you control which parts of the model are part of the gradient computation.
Because this is such a common pattern, `requires_grad` can also be set at the module level with `nn.Module.requires_grad()`. When applied to a module, `requires_grad()` takes effect on all of the module's parameters.

#### No-grad Mode
Computations in no-grad mode behave as if none of the inputs require grad. In other words, computations are never recorded in the backward graph.

> [!INFO]
> use no-grad mode when you need to perform operations that shouldn't be recorded by Autograd, but you'd still like to use the outputs of these computations with Autograd later on. 


#### Inference Mode
Inference mode is an extreme version of no-grad mode.
They're similar in that computations are not recorded in the backward graph, but enabling inference mode will allow PyTorch to speed up your model even more.
This better runtime comes with a drawback: tensors created in inference mode will not be able to be used in computations to be recorded by Autograd after exiting inference mode.

> [!INFO]
> Use inference mode when doing computations that don’t need to be recorded in the backward graph, AND you don’t plan on using the tensors created in inference mode in any computation that is to be recorded by Autograd later.


#### Eval Mode

Eval mode isn't actually a mechanism to locally disable gradient computation. It's included here anyway because it's often confused to be one.

Functionally, `module.eval()` is completely orthogonal to no-grad and inference mode. How it affects your model depends entirely on the specific modules used in the model and whether they define any training-specific behaviour.

Two examples of modules that differ between train and test are `nn.Dropout` and `nn.BatchNorm2d`.



