These notes are based on this [EleutherAI blog](https://blog.eleuther.ai/rotary-embeddings/), and the [original RoPE paper](https://arxiv.org/pdf/2104.09864.pdf).

# Before We Being: How SMX's Rotary Embeddings are Different
The rotary embeddings used in our TXL are different to the implementation described in this blog. There are 3 ways in which this is the case:

1. Length of key $\not =$ Length of query. But we want to apply the same rotation to the corresponding locations in the $\boldsymbol{K}$ and $\boldsymbol{Q}$ vectors. So we use an offset (that's equal to the length of the state). David has written a[ Confluence page](https://speechmatics.atlassian.net/wiki/spaces/~709818302/pages/3431333891/Rotary+embedding+offset) explaining this.
2. We cache the cos & sin needed to compute the embeddings, and never recompute them. This means we have a fixed cache, which cannot be increased in length. This is fine at training (where we have a fixed context and window size). At inference, we support arbitrarily long window lengths. So there would be a failure if the window length exceeded this maximum (somewhere over 10 seconds).
3. We apply a *partial* rotary embedding. This means that the rotary embedding is only applied to a subset of the features of the $\boldsymbol{K}$ and $\boldsymbol{Q}$ vectors.

```toc
```

# Positional Embeddings used in *Attention is All You Need*
In [Attention is All You Need](Attention%20is%20All%20You%20Need.md), *absolute* positional information is added to the input tokens. This is done by summing the query and key vector with a sinusoidal positional encoding. They use:

![](_attachments/Screenshot%202022-08-25%20at%2022.54.55.png)

where $pos$ is the position of the vector, and $i$ is the dimension. In other words, each dimension of the positional encoding corresponds to a sinusoid. The wavelengths form a geometric progression from $2\pi$ to $10000 \cdot 2\pi$. 

These positional embeddings are transferred through the network via the residual connections.

## Intution: How This Encodes Absolute Positional Information
It's not immediately obvious how this encodes absolute positional information. The key idea is that the positional information is encoded in the *difference* between different elements of the embedding. In our case, the wavelength of the added value varies along the vector axis.

To understand this, let's consider binary numbers. Counting from 0 to 15:

![](_attachments/Screenshot%202022-08-26%20at%2015.23.39.png)

We can see that the rate of change between the least significant bit has highest frequency, the leftmost has the lowest frequency, and so on.

The exact same logic applies to our positional embeddings. Instead of working with (discrete) binary values, we work with continuous values. The wavelengths of different sinusoids mean we can infer absolute positional information:

![](_attachments/Screenshot%202022-08-26%20at%2015.25.45.png)

In this image, we can imagine the positional embedding for a specific position lying along the horizontal axis.

# What's the Problem?
Since Vaswani et al., 2017 there have been many schemes for encoding positional information in transformers. Prior to Rotary Embeddings (RoPE), all existing methods had limitations. For instance, *learned* absolute positional encoding are simple, but may not generalise and are not always meaningful due to the common practices of packing short sentences and phrases together in a single context and breaking up sentences across contexts.

RoPEs unify both absolute and relative positional approaches.

# What's the Solution?
## Intuition
We would like to find a positional encoding function $f(\boldsymbol{x}, l)$ for a vector $\boldsymbol{x}$ and position $l$ such that, for two items $\boldsymbol{K}$ and $\boldsymbol{Q}$ at positions $m$ and $n$, the inner product between $f(\boldsymbol{K}, m)$ and $f(\boldsymbol{Q}, n)$ is sensitive only to the values of $\boldsymbol{K}$, $\boldsymbol{Q}$, and their relative position $m-n$. 

A key piece of information is that the dot product between two vectors is a function of the magnitude of the individual vectors, and the angle between them: $\boldsymbol{Q} \cdot \boldsymbol{K} = |\boldsymbol{Q}||\boldsymbol{K}|\cos(\theta_{QK})$ .

With this in mind, the intuition behind RoPE is that we can represent token embeddings as complex numbers, and their positions as pure rotations that we apply to them. The subsequent inner product used in self-attention will have the property we are looking for - relative positional information.

To be clear: the RoPE function $f(\boldsymbol{x}, l)$ contains *absolute* positional information; the subsequent inner product will contain *relative* positional information.

The following is an example illustrating the core idea of RoPE. Some arbitrary $0<\epsilon<\frac{\pi}{2N}$ is chosen, where $N$ is the maximum sequence length. When viewed elementwise on $\boldsymbol{q}$ and $\boldsymbol{k}$, with $j$ as the element index, RoPE can be viewed as:

![](_attachments/Screenshot%202022-08-25%20at%2012.00.26.png)

The value $\epsilon$ varies along the dimension axis, as in Vaswani et al. Again, this is how absolute positional information is encoded.

## Derivation
We begin with *absolute* positional information: for each token, we know where it is in the sequence. However, dot products (and therefore attention) do not preserve absolute positional information. So if we encode that positional information in the absolute position of the embeddings (in their embedding space), we will lose a significant amount of information.

On the other hand, dot products do preserve relative position. So **if we can encode the *absolute* positional information into the token embeddings in a way that only leverages *relative* positional information**, that will be preserved by the attention function.

While it's common in ML to restrict our focus to real numbers, for rotary embeddings it is mathematically more convenient to use the complex numbers as the base field for our space. Instead of working in $\mathbb{R}^d$, we will work in $\mathbb{C}^{d/2}$ by considering consecutive pairs of elements of the query and key vectors to be a single complex number. Specifically, instead of viewing $\boldsymbol{q}=(q_1,q_2,\dots,q_d)$ as a $d$-dimensional real vector, we view it as $\boldsymbol{q}=(q_1+iq_2,\dots,q_{d-1}+iq_d) \in \mathbb{C}^{d/2}$.
If $d$ is odd, we can pad it with a dummy coordinate to ensure things line up correctly. Alternatively, we can simply increase $d$ by one.

Let $\boldsymbol{q}$ and $\boldsymbol{k}$ be query and key vectors respectively. Let $m$ and $n$ be the *absolute* positions of the corresponding tokens. Let $f(\boldsymbol{x}, l)$ be the function that takes the token embedding $\boldsymbol{x}$ in position $l$ and outputs a new embedding that contains (in some fashion) the *relative* positional information. Our goal is to find a function $f$ for which the inner product encodes position information only in the relative form:

![](_attachments/Screenshot%202022-08-25%20at%2012.07.34.png)

where $g(\boldsymbol{q}, \boldsymbol{k}, m-n)$ now represents the pre-softmax logit of the usual attention equation. Writing these in exponential form gives:

![](_attachments/Screenshot%202022-08-25%20at%2022.06.43.png)

where $R_f$ and $\Theta_f$ are the radial and angular components of $f$. Computing the inner product and equating corresponding components yields:

![](_attachments/Screenshot%202022-08-25%20at%2022.09.38.png)
We can now apply some boundary condition. We have an initial condition that $f(\boldsymbol{x}, 0) = \boldsymbol{x}$, which we can use this by setting $m=n$:

![](_attachments/Screenshot%202022-08-25%20at%2022.12.13.png)

This equation holds for all $m$. Which means we can set $R_f(\boldsymbol{x}, y)=\boldsymbol{x}$. We can use a similar trick for $\Theta_f$:

![](_attachments/Screenshot%202022-08-25%20at%2022.16.19.png)

which implies that $\Theta_f(\boldsymbol{q},m)-\Theta(\boldsymbol{q},0)=k$, where $k$ is a constant, for all $\boldsymbol{q}, m$. This allows us to decompose $\Theta_f$ as $\Theta_f(\boldsymbol{x},y)=\Theta(\boldsymbol{x}, 0)+\psi(y)$. Examining the case of $m=n+1$ reveals:

![](_attachments/Screenshot%202022-08-25%20at%2022.22.41.png)

where $\Theta(\boldsymbol{q})=\Theta(\boldsymbol{q},0)$. Setting $\Psi(0)=0, \Psi(1)=\theta$ gives $\Psi(m)=m\theta$.

Putting all these pieces together, we have the final formula for the rotary positional embedding:

![](_attachments/Screenshot%202022-08-25%20at%2022.26.10.png)

and likewise for $\boldsymbol{l}$. A reminder that $j$ corresponds to an element of our vector $\boldsymbol{q}$.

This can be written in matrix form as:

![](_attachments/Screenshot%202022-08-25%20at%2022.31.58.png)

where $\boldsymbol{\Theta}_m$ is the block diagonal rotation matrix, $\boldsymbol{W}_q$ is the learned query weights, $\boldsymbol{X}_m$ is the embedding of the $m$ token, and $M_j$ is:
![](_attachments/Screenshot%202022-08-25%20at%2022.32.39.png)
Note that $M_1$ acts on $q_1$ and $q_2$, $M_2$ acts on $q_3$ and $q_4$, etc.

## Visualisation
What is this actually doing? It turns out the picture is pretty simple. Let's consider the case for a single embedding. It is converted, via a linear transformation, into a query vector (in this example). Each *pair* of coordinates in this query vector then undergoes a rotation about the origin in a 2D vector space. This is summarised nicely in Figure 1 of the RoPE paper:

![](_attachments/Screenshot%202022-08-25%20at%2022.42.21.png)

An important point is that the degree to which each pair of coordinates is rotated ($\theta_i$) is different between each pair of coordinates. The formula to calculate this is given in the original paper as $\theta_i=10000^{-2i/d}$ (a reminder that $d$ is the dimensionality of the query/key vectors). This is the same as that used in the original [Vaswani paper](Attention%20is%20All%20You%20Need.md).

The authors of the RoPE paper explain that this setting provides a long-term decay property, whereby the inner product will decay when relative distance increases. Being honest, I am not sure how this follows.

## How is this Different to the Positional Embeddings used in *Attention is All You Need*?
There are three ways (that I can think of) that RoPEs are different:

1. Sinusoidal embeddings apply to each coordinate individually, while RoPEs mix pairs of coordinates;
2. Sinusoidal embeddings are additive, while RoPEs use a multiplicative factor.
3. **RoPEs have to be applied at every layer in the transformer,** whereas additive positional embeddings do not. The reason for this is that Vaswani positional embeddings are added to the input embeddings, whereas RoPEs are applied to the *query/key* vectors.

# Things I Don't Understand
A few parts that are unclear:

1. I understand how the Vaswani positional embeddings encode absolute positional information. But how can a neural network extract positional information from the summation of two vectors?
2. How is it that setting $\theta_i=10000^{-2i/d}$ implies a long-term decay property?
3. RoPEs are described as unifying absolute and relative positional information. But any absolute positional information encoded in the Q/K vectors is lost after computing the dot product. So what's the point?
