I realised I don't *really* get how RNNs work. So I made some quick notes to cover this.
Most of the content is based on [Chapter 10 of the Deep Learning Book](https://www.deeplearningbook.org/contents/rnn.html).

```toc
```

## Overview
RNNs are a family of neural nets for processing sequential data.
We consider RNNs as operating on a sequence that contains vectors $\boldsymbol{x}^{(t)}$ with time step index $t$ ranging from $1$ to $\tau$.

## Unfolding Computational Graphs
We can unfold a recurrent computation into a computational graph that has a repetitive structure. For instance, consider the following recurrence relation:

$$\boldsymbol{s}^{(t)}=f(\boldsymbol{s}^{(t-1)}, \boldsymbol{\theta})$$

This can be drawn as a traditional DAG:

![](_attachments/Screenshot%202023-02-06%20at%2019.41.45.png)

Now consider some dynamical system driven by an external signal, $\boldsymbol{x}$:

$$\boldsymbol{s}^{(t)}=f(\boldsymbol{s}^{(t-1)}, \boldsymbol{x}^{(t)},\boldsymbol{\theta})$$

The state contains information about the whole past sequence of $\boldsymbol{x}$. 
Many RNNs use a similar equation to the following to define values of their hidden units:

$$\boldsymbol{h}^{(t)}=f(\boldsymbol{h}^{(t-1)},\boldsymbol{x}^{(t)}, \boldsymbol{\theta})$$

where we use the $h$ symbol to indicate that the state is the hidden units of the network. Typical RNNs will add output layers that read information out of $\boldsymbol{h}$ to make predictions.

This equation can be drawn as another DAG:

![](_attachments/Screenshot%202023-02-06%20at%2019.47.10.png)

(We use a black square in a circuit diagram to indicate that an interaction takes place with a delay of a single time step).

This diagram shows both the recurrent graph and the unrolled graph. Both have their advantages: the recurrent graph is succinct, whilst the unfolded graph provides an explicit description of which computations to perform.

> [!INFO]
> When an RNN is trained to perform a task that requires predicting the future from the past, the network typically learns to use $\boldsymbol{h}^{(t)}$ as a lossy summary of the task-relevant aspects of the past sequence of inputs.

## Recurrent Neural Nets

We can now imagine an RNN that produces an output at each time step:

![](_attachments/Screenshot%202023-02-06%20at%2019.58.02.png)

where a loss $L$ measures how far output $\boldsymbol{o}$ is from target $\boldsymbol{y}$.

We can write down the equations that govern this network. We begin with an initial state $\boldsymbol{h}^{(0)}$  and then, for each time step, do:

![](_attachments/Screenshot%202023-02-06%20at%2020.01.42.png)

where the learnable parameters are weight matrices $\boldsymbol{W}, \boldsymbol{U}, \boldsymbol{V}$ and biases $\boldsymbol{b}, \boldsymbol{c}$. 
The total loss for a given sequence of $\boldsymbol{x}$ values paired with a sequence of $\boldsymbol{y}$ values would then be just the sun of losses over all time steps.

Doing a forwards and backward pass with this model is an expensive operation. The runtime is $O(\tau)$ and cannot be reduced by parallelization because the forwards propagation graph is inherently sequential: each time step can only be computed after the previous one.

## Implementation of a Really Simple RNN
I modified the PyTorch code from [this blog](https://medium.com/@VersuS_/coding-a-recurrent-neural-network-rnn-from-scratch-using-pytorch-a6c9fc8ed4a7) for an RNN with a single layer. It corresponds to the equations for an RNN, given above. 

```python
class RNN(nn.Module):  
    
    def __init__(self, input_size, hidden_size, output_size):  
        super().__init__()  
        self.hidden_size = hidden_size  
        self.i2h = nn.Linear(input_size, hidden_size, bias=False)  
        self.h2h = nn.Linear(hidden_size, hidden_size)  
        self.h2o = nn.Linear(hidden_size, output_size)        
    
    def forward(self, x, hidden_state)  
        x = self.i2h(x)  
        hidden_state = self.h2h(hidden_state)  
        hidden_state = torch.tanh(x + hidden_state)  
		out = self.h2o(hidden_state)  
        return out, hidden_state    
        
    def init_state(self):  
        return torch.zeros(self.hidden_size, requires_grad=False)
```

To run this model, we'd do something like:

```python
outputs = []
state = model.init_state()
for data in dataset:
    output, state = model(data, state)
    outputs.append(output)
```

## Deep RNNs
The computation in RNNs can be decomposed into three blocks:

1. Input $\to$ hidden state
2. Hidden state $\to$ hidden state
3. Hidden state $\to$ output

With our above architecture, each of these is associated with a single weight matrix. But it can be advantageous to add depth to these transformations. 

There are multiple approaches to this:

> a. Decompose the state into multiple layers
> b. Use deeper computation for each of the input-to-hidden, hidden- to-hidden, and hidden-to-output parts
> c. Do (b), but add skip connections (c.f. [transformers](Transformers/Attention%20is%20All%20You%20Need.md))

These three approaches are shown here:

![|500](_attachments/Screenshot%202023-02-10%20at%2018.04.25.png)

## The Challenge of Long-Term Dependencies
RNNs struggle to learn long-term dependencies. The basic problem is that gradients propagated over many stages tend to either vanish or explode. Even if we assume that the network training is stable, the difficulty arises from the exponentially smaller gradients associated with long-term interactions.

We can also understand this by considering a forwards pass. Suppose we have a super-simple RNN with no activation function, and no inputs:

$$\boldsymbol{h}^{(t)}=\boldsymbol{W}^T\boldsymbol{h}^{(t-1)}$$

This can be written as

$$\boldsymbol{h}^{(t)}=(\boldsymbol{W}^t)^T\boldsymbol{h}^{(0)}$$

and if $\boldsymbol{W}$ admits an eigendecomposition of the form:

$$\boldsymbol{W}=\boldsymbol{Q}\boldsymbol{\Lambda}\boldsymbol{Q}^T$$

then the recurrence relation can be written as

$$\boldsymbol{h}^{(t)}=\boldsymbol{Q}^T\boldsymbol{\Lambda}^t\boldsymbol{Q}\boldsymbol{h}^{(0)}$$

All eigenvalues are raised to the power of $t$, causing exponential decay/explosion. Any component of $\boldsymbol{h}^{(0)}$ that is not aligned with the largest eigenvector will eventually be discarded.

This problem is particular to RNNs. 









