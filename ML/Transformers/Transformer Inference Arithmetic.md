These notes are based upon [this Kipply blog](https://kipp.ly/blog/transformer-inference-arithmetic/). It presents simple reasoning about LLM inference performance, with no experiments or difficult maths.

> [!INFO]
> While there is loads of super interesting stuff here, I found the blog wasn't amazingly well-written. I've tried to get the best bits, and have skipped others. Any sections that I skipped have been left blank in these notes.

```toc
```

## KV Cache
* When sampling, self-attention requires the KV values for each item in the sequence.
* These vectors are provided in a matrix called the "kv_cache".
* It would be shaped like `[batch, 2, num_heads, seq_len, features]`.
* The purpose of this is to avoid recalculations of those vectors every time we sample a token. We save computation at the cost of storage. 
* Per token, the number of bytes we store is:

$$2\cdot2\cdot n_\textrm{layers} \cdot n_\textrm{heads} \cdot d_\textrm{head}$$
* The first factor of 2 accounts for K & V. The second factor assumes FP16.
* How many FLOPS do we require to compute the K & V for all our layers (for an individual token)?
* The weights that we multiply by the token embeddings are $W_k,W_V\in \mathbb{R}^{d_{\textrm{model}}\times d_{\textrm{model}}}$, and each token embedding is $t_e\in\mathbb{R}^{1\times d_{\textrm{model}}}$.
* We multiply each $t_e$ by $W_k$, which takes $2\cdot d_{\textrm{model}}^2$ FLOPS.
* So the total is:

$$2\cdot2\cdot n_\textrm{layers} \cdot d_{\textrm{model}}^2$$

> [!INFO]
> The computation for a matrix-vector multiplication is $2mn$ for $A\in\mathbb{R}^{m\times n}$, $b\in\mathbb{R}^{n}$. A matrix-matrix multiplication is $2mnp$. These [lecture notes](https://www.stat.cmu.edu/~ryantibs/convexopt-F18/scribes/Lecture_19.pdf) have more detail.

> [!INFO]
> **On Memory Boundedness**
> To do computations, loading the weights costs memory bandwidth. Computations begin as we load the weights.

* An A100 does `312e12` FLOPs per second, and `1.5e12` bytes per second of memory bandwidth. 
* The times spent on data transfer and compute are:

$$\begin{equation}
\textrm{Time to load KV weights}=2\cdot2\cdot n_\textrm{layers} \cdot d_{\textrm{model}}^2 \div 1.5\textrm{e}12
\end{equation}$$
$$\textrm{Time to compute KV vectors for one token}=2\cdot2\cdot n_\textrm{layers} \cdot d_{\textrm{model}}^2 \div 312\textrm{e}12$$

* So the ratio is always `208`. Loading KV weights takes 208x more time than computing the KV vectors for one token!
* This means that computing KV for one token takes the same time as computing KV for 208 tokens.
* For fewer than 208 tokens, and we're memory bandwidth bound. Above, we're FLOPS bound.
* If we consider the entire forwards pass (besides just the KV calculation), the ratio is still 208 (both the numerator and denominator get a [factor of 6](#Flops%20Counting) added).
* The result is this diagram below. It shows the time taken to load our model weights, and do compute, for varying batch sizes.
* It has an intersection at 208.

![](_attachments/Screenshot%202023-04-19%20at%2017.09.14.png)

> [!QUESTION]
> I understand this diagram somewhat. It says that the time spent doing a forward pass for $n$ tokens is equal to the time spent loading the entire model weights when $n=208$. What I don't get is *exactly* how it relates to the KV cache.

* To be clear, the memory requirements are constant, since we always load the same weight matrices, regardless of batch size.
* Calculating the KV cache for a token is exactly [1/6th](#Flops%20Counting) of the compute of passing the token through the model.  
* This means the benefit of using a KV cache is *even more significant*:
	* Don't use KV cache $\implies$ have to do compute for entire model, for every token.
	* Do use KV cache $\implies$ only have to load KV vectors.
* Furthermore, without a KV cache, sampling would be quadratic in time complexity - we'd have to compute the KV vectors for 1 vector, then 2, then 3, ...
* This is not the whole story. There are some overheads and tradeoffs associated with storing this cache. If we're serving small batches, we may be memory bandwidth bound rather than flops bound.

## Capacity
* We store two things in our GPUs - kv cache, and weights.
* Let's evaluate how GPU capacity comes into play for transformer performance at inference.
* Nvidia A100s have a standard of 40GB capacity in DRAM.
* Given the parameter count, we can multiply by 2 to get bytes. The size of weights for a 52B model is:
$$52\textrm{e}9\cdot 2=104\textrm{e}9 \space \textrm{bytes}\approx 104 \space \textrm{GB}$$
* This won't fit on one GPU!
* If we use 3 GPUs, that leaves us with $120-104=16GB$ for the KV cache.
* How many tokens is that good for? Again, with a 52 bn parameter model:

![|400](_attachments/Screenshot%202023-04-19%20at%2017.39.04.png)

* So we could fit $16/0.002\approx 8000$ tokens' worth of KV cache with this GPU setup.
* Or we could use a batch size 4 where each request has up to 2048 tokens.
* This sucks - we'd like to be able to do higher batch sizes (since this guarantees we are compute-bound - see the graph above), but are capacity limited!
* At batch sizes this low, we're gonna be memory bound, and should forgo the KV cache, and just pay the flops cost instead.

## Model Parallelism
* The outcome of model parallelism is that the cost of all memory transfers and FLOPs are all divided by the "degree" (the number of accelerators[^fn1] the model is split across)[^fn2]. 
* We assume [Tensor Parallel](../Tensor%20Parallel.md). 
> [!INFO]
> #### An Aside on Pipeline Parallel
>  * A more naive approach is pipeline parallel, where each GPU holds a fraction of the layers. This evens out the weight loading costs, just like tensor parallel.
>  * In practice, you could pipeline through different batches (as the first batch moves onto the next GPU, start a new batch on the first GPU). But this doesn't work for a single request.
>  * Pipeline also doesn't exhaust the memory bandwidth.
>  * The only place it does better is communication. A pipeline parallel model would communicate $d_{\textrm{model}}$ per accelerator, while tensor parallel does $N\cdot d_{\textrm{model}}$ per layer, where $N$ is the number of accelerators.


> [!QUESTION]
> Why does pipeline parallel not exhaust the memory bandwidth?

* A100s have a communication bandwidth of 300 GB/s. Nvidia quotes 600 GB/s, as it's adding 300 GB/s both in and out of the chip simultaneously.
* Consider this diagram:

![](_attachments/Screenshot%202023-04-19%20at%2017.51.14.png)

* We start by inserting our token embeddings into the bottom of the model.
* The purple boxes outline how weights are split across accelerators.
* For attention:
	* The parallelism is intuitive since we've multiple heads. 
	* The QKV matrices are split column-wise (upper left purple box).
	* After multiplying by $v$, we multiply by a row-wise shard of $W_o$. I think this is a mistake in the above diagram.
	* We must then do an AllReduce (specifically, a sum) of a $d_\textrm{model}$ vector across all GPUs.
	* Each accelerator does an "even share" of the addition, and then each host does its own concatenation of the results.
	* Each accelerator receives + sends a $\frac{d_\textrm{model}}{N}$ vector $N-1$ times.

> [!QUESTION]
> I'm not sure on exactly which AllReduce algorithm that Kipply is thinking of here. The $(N-1)\cdot\frac{d_\textrm{model}}{N}$ factor seems correct for a [Ring-Based AllReduce](../../Computing/Ring-Based%20AllReduce.md), although I'd have personally added an extra factor of $2$ to account for the share-reduce and share-only phases.

* The MLP layer is similar. The FF1 weights are split column-wise, and the FF2 weights are split row-wise.
* Ultimately, we do $4\cdot(N-1)\cdot\frac{d_\textrm{model}}{N}$ bytes of communication. I think this factor of $4$ is due to $2$ bytes per element, and a communication happening in the Attention and MLP blocks.
* The KV cache is split across heads by GPU.

## Latency Calculations
* Our latency calculations are mostly about FLOPs vs memory boundedness.




## Batch Size

## Flops Counting
* This section justifies the claim that the KV calculations are 1/6 of the whole compute for a forwards pass.
* The following calculations are for a single token, and a single layer. 
* Assume $W_q,W_k,W_v\in\mathbb{R}^{d_{\textrm{model}}\times d_{\textrm{model}}}$.
* Then we have the following compute costs:

| Stage | FLOPs  
| ---------- | --------- |
| Q calculation| $O(2\cdot d_{\textrm{model}}^2)$ |
| K calculation| $O(2\cdot d_{\textrm{model}}^2)$ |
| V calculation| $O(2\cdot d_{\textrm{model}}^2)$ |
| Softmax| $O(d_{\textrm{model}})$ |
| Final Projection | $O(2\cdot d_{\textrm{model}}^2)$ |
| Feed-forward 1 | $O(2\cdot 4\cdot d_{\textrm{model}}^2)$ |
| ReLU | small |
| Feed Forward 2| $O(2\cdot 4 \cdot d_{\textrm{model}}^2)$ |
| **Sum**| $O(2\cdot 12\cdot d_{\textrm{model}}^2)$ |

* Hence the fraction of compute spent on KV is $\frac{2\cdot 2 \cdot d_{\textrm{model}}^2}{2\cdot12 \cdot d_{\textrm{model}}^2}=\frac{1}{6}$.
* Note that we've ignored a few other things, such as:
	* Layernorm
	* Embedding & unembedding matrices
	* Positional embeddings
## Intermediate Memory Costs



[^fn1]: GPUs or TPUs
[^fn2]: Suppose a model fit on 2 GPUs, but was used in combination with data parallel by using 20 GPUs in total. In this case, the degree of model parallelism is 2, and the degree of data parallelism is 10.











