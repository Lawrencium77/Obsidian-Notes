I made a few notes on how the embedding layer works in [GPT](GPT-3.md)-style [transformers](Attention%20is%20All%20You%20Need.md). I found the [Logit Lens](Logit%20Lens.md) blog very helpful for understanding this.

In practice, embedding layers can consume large amounts of memory. They can therefore require [vector quantization](Vector%20Quantization.md).

```toc
```

### From Tokens to Embeddings
As input, GPT takes a sequence of discrete tokens. Each of these can be represented as a one-hot vector of size $N$, where $N$ is the vocab size.
The first thing that happens is a multiplication by an embedding matrix, $W \in \mathbb{R}^{N\times d}$, where $d$ is the feature dim of our neural net.

This looks something like:

![](_attachments/Screenshot%202023-01-10%20at%2019.25.28.png)

We can think of the matrix $W$ as a **dictionary** that lets us convert from token space to embedding space. To get the embedding for a token, we simply **look up** the corresponding entry in $W$.

Another way of interpreting the embedding function is as a [hash function](../../Computing/Hash%20Maps.md).

## From Embeddings to Tokens
There isn't much complicated in any of this. The only thing to be aware of is that at the end of the network, our $d$-dimensional embedding is multiplied by $W^T$ to project back into vocab space.
The resulting vector is treated as logits. We apply a **softmax** to convert these to probabilities.

> [!IMPORTANT]
> To convert from tokens to embeddings, we multiply by $W$.
> To convert from embeddings to tokens, we multiply by $W^T$.
> We use the **same parameters** for both operations.

Here's a diagram:

![](_attachments/Screenshot%202023-01-10%20at%2019.32.28.png)

An intuition for this final layer is that each logit measures how "similar" our output embedding is to each of the token embeddings in our vocabulary.