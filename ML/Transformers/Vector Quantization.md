```toc
```

## Vector Quantization of Embedding Layers
With large vocabulary sizes, [transformer](Attention%20is%20All%20You%20Need.md) [embedding layers](On%20The%20GPT%20Embedding%20Layer.md) consume substantial memory. To mitigate this we can use vector quantization (cf. [the other type of quantization](../../Computing/Quantisation/Quantization%20in%20Neural%20Nets%20-%20Key%20Concepts.md)). 

Originally introduced in [this paper](https://lear.inrialpes.fr/pubs/2011/JDS11/jegou_searching_with_quantization.pdf), the key idea is that instead of maintaining the entire embedding matrix $W$, we can approximate using clustering.

Let's consider the simplest case. Without vector quantization, we have an embedding matrix $W$ of shape $D \times N$ (this is different to above but it doesn't matter):

![|500](_attachments/Screenshot%202023-01-13%20at%2018.37.29.png)

In the simplest case, vector quantization approximates $W$ by mapping **groups** of input tokens to the same embedding vector. This means we have a new matrix of size $D \times K$ (where $K < N$):

![|500](_attachments/Screenshot%202023-01-13%20at%2018.42.45.png)

The tensor $C$ is usually called a **codebook**. 
This is just a form of **clustering**. Groups of "similar" embeddings are mapped to a single centroid. The locations of these centroids might be learned with $K$-means.

### Product Quantization

In **product quantization**, we split each vector $\boldsymbol{x} \in \mathbb{R}^D$ into $m$ subvectors, each of dimension $D/m$. Each subvector is quantized using its own distinct quantizer.

We use $K^* < K$ centroids for each subspace. Graphically:

![|500](_attachments/Screenshot%202023-01-15%20at%2017.42.22.png)

The benefit is that we produce a large set of centroids from several small sets. The total number of centroids is $(K^*)^m$. 
As in standard vector quantization, the centroids within each subspace are learned with $K$-means. 