These notes are part of a [series](Summary.md) I made about automatic differentation. They are based on [this blog](https://sidsite.com/posts/autodiff/).

```toc
```

## How does Autodiff Work?
Let's start with an example:

![](_attachments/Screenshot%202022-08-29%20at%2011.33.41.png)

What is the gradient of $\frac{\partial d}{\partial a}$? Since $d=d(a,c)=d(a,c(a,b))$, the chain rule tells us:

$\frac{\partial d}{\partial a}=\frac{\partial d}{\partial a}\frac{\partial a}{\partial a}+\frac{\partial d}{\partial c}\frac{\partial c}{\partial a}=\frac{\partial d}{\partial a}+\frac{\partial d}{\partial c}\frac{\partial c}{\partial a}=c+a\cdot 1 = 7 + 4 = 11$

### Solving the Autodiff Way
We can represent this computation as a graph (left):

![](_attachments/Screenshot%202022-08-29%20at%2011.37.33.png)

Each variable is a node, e.g. $d$ is the topmost node and $a$ and $b$ are leaf nodes at the bottom.

On the right, we see the system from autodiff's perspective. We can call values on the graph edges **[local derivatives](#^54f305)**. We see that we can get the derivative $\frac{\partial d}{\partial a}$ by finding the paths from $d$ to $a$, and then applying the following rules:

* Multiply the edges of a path
* Add together different paths

The rationale for these is pretty obvious, given the chain rule.

> [!INFO]
> The terms of the form $\frac{\partial \bar{y}}{\partial x}$ in the above figure are the [adjoint](What%20is%20Autodiff?#^0f7290). They are sometimes called the **local derivative**.


## A Simple Python Implementation

### Description 

A variable (or node) consists of two pieces of data:

1. `value` - the value of the variable
2. `local_gradients` - the variable's children and corresponding local derivatives

Here's a simple example:

```python
class Variable:
   def __init__(self, value, local_gradients=()):
      self.value = value
      self.local_gradients = local_gradients

def add(a, b):
	"""
	Create the variable that results from adding two variables
	"""
	value = a.value + b.value
	local_gradients = (
		(a, 1), # Local derivative wrt a is 1
		(b, 1)  # Local derivative wrt b is 1
	)
	return Variable(value, local_gradients)
	

def mul(a, b):
	"""
	Create the variable that results from multiplying two variables
	"""
	value = a.value * b.value
	local_gradients = (
		(a, b.value), # Local derivative wrt a is b.value
		(b, a.value)  # Local derivative wrt b is a.value
	)
	return Variable(value, local_gradients)
```

One way of conceptualising this is that we're creating a computation graph for our backwards pass every time that we do a forwards pass. This is exactly [what PyTorch does](PyTorch%20Autograd%20-%20More%20Detail.md). 

The function `get_gradients` uses the variables' `local_gradients` data to go through the graph recursively, computing the gradients. In other words, `local_gradients` contains references to child variables, which have their own `local_gradients`, which contain references to other child variables, etc.

The gradient of `variable` with respect to a child variable is computed using the rules we saw above:

* For each path from `variable` to the child variable, multiply the edges of the path (giving `path_value`).
* Sum the path values

Here is the code for `get_gradients`:

```python
def get_gradients(variable):
   """
   Compute the first derivative of `variable` wrt child variables
   """
   gradients = defaultdict(lambda: 0)

   def compute_gradients(variable, path_value):
      for child_variable, local_gradient in variable.local_gradients:
         # Multiply the edges of a path:
         value_of_path_to_child = path_value * local_gradient
         # Add together the different paths:
         gradients[child_variable] += value_of_path_to_child
         # Recurse through graph:
         compute_gradients(child_variable, value_of_path_to_child)

	compute_gradients(variable, path_value=1)
	return gradients
```

> [!INFO]
> Another way of interpreting this is that we build a DAG representing our backwards pass and then do a depth-first traversal to compute gradients. PyTorch doesn't actually do this. Instead, it uses a "tape" mechanism. My current theory is that this enables a breadth-first search, which is more compatible with [gradient checkpointing](DDP.md#^b60279).

### Using it to Solve Our Example

To solve our above example, we define the computational graph and then use `get_gradients`:

```python
a = Variable(4)
b = Variable(3)
c = add(a,b) # = 4 + 3 = 7
d = mul(a,c) # = 4 * 7 = 28

gradients = get_gradients(d)

print(f"d.value = {d.value}")
print(f"The partial derivative of d wrt a =  = {gradients[a]}")
```

This gives:

```
d.value = 28
The partial derivative of d wrt a = 11
```

Note that we also get the gradients for other nodes:

```python
print(f"gradients[b] = {gradients[b]}")
print(f"gradients[c] = {gradients[c]}")
```

```
gradients[b] = 4
gradients[c] = 4
```

> [!INFO]
> The [blog post](https://sidsite.com/posts/autodiff/) gives a few more improvements to this starting point. But they key ideas are those shown above.

## Nth Order Derivatives
Enabling Nth order derivatives using reverse-mode autodiff is more computationally costly that only enabling first order derivatives. 
As a result, most deep learning frameworks only compute first order derivatives. 

We can enable our framework to compute Nth order derivatives by changing the `get_gradient` computations to use our `Variable` objects. This means that autodiff graphs will be created when computing derivatives, and we can then compute the derivatives of the derivatives (and so on...).

> [!TODO]
> Complete this section of the notes.




