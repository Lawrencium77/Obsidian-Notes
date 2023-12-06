These notes provide a summary of Flash Attention. They're based on [this](https://arxiv.org/pdf/2205.14135v2.pdf) paper, and the [Triton lang implementation](https://arxiv.org/pdf/2205.14135v2.pdf).

```toc
```

## Intro 
[Transformers](../ML/Transformers/Attention%20is%20All%20You%20Need.md) are memory-hungry on long sequences, since the time and memory complexity of self-attention are quadratic in sequence length. Approximate attention methods have attempted to address this by trading off model quality to reduce complexity, but rarely achieve a speedup in practice.
 
With **FlashAttention**, we focus on reducing the number of read/writes between GPU SRAM and DRAM,  since most operations on GPU are bottlenecked by memory accesses.

The main goal is to avoid reading/writing the whole attention matrix to and from DRAM. This requires:
* Computing the softmax reduction without access to the whole input
* Not storing the large intermediate attention matrix for the backwards pass

To achieve these, we employ two pretty common tricks:
* **Tiling**
* **Recomputation** of the attention (on-chip) during the backwards pass.

Their algorithm uses **less memory** and **runs faster** due to the reduced number of DRAM accesses. They also point out that FlashAttention could potentially make approximate attention algorithms more useful by overcoming their issues with memory access overhead.

To be clear: the authors are arguing that the main determining factor of attention run-time is HBM accesses. Even though FlashAttention has higher FLOP count compared to standard attention (due to recomputation in the backward pass), it has much fewer HBM accesses, resulting in much faster runtime.

## Background - Standard Attention Implementation
Given input sequences $\boldsymbol{Q},\boldsymbol{K},\boldsymbol{V} \in \mathbb{R}^{N\times d}$, where $N$ is sequence length and $d$ is the head dimension, we want to compute the attention output $\boldsymbol{O}\in\mathbb{R}^{N\times d}$:

![](_attachments/Screenshot%202022-11-03%20at%2017.34.59.png)

where the softmax is applied row-wise (i.e. across keys for a single query).
The standard attention implementation is shown below:

![](_attachments/Screenshot%202022-11-03%20at%2017.37.51.png)

This writes out both $\boldsymbol{S}$ and $\boldsymbol{P}$ to DRAM, since they are required for the backwards pass. A lot of the operations are memory-bound (e.g. softmax), meaning that the large number of memory accesses translates to a slow wall-clock time.

## FlashAttention - Algorithm
Flash Attention uses fewer DRAM reads/writes, and doesn't store large intermediate matrices $\boldsymbol{S}$ and $\boldsymbol{P}$ for the backward pass.
Here, we focus only on the forward pass.

The main idea is to split the inputs $\boldsymbol{Q},\boldsymbol{K},\boldsymbol{V}$ into blocks, load them into SRAM, and then compute attention with respect to those blocks. By scaling the output of each block by the right normalisation factor before adding them up, we get the correct result.

Let's first look at the algorithm in full:

Let's go step-by-step. We first split the inputs $\boldsymbol{Q},\boldsymbol{K},\boldsymbol{V}$ into blocks (line 3):

![](_attachments/Screenshot%202022-11-06%20at%2013.54.11.png)

We choose $B_c = \lfloor \frac{M}{4d} \rfloor$ since we have SRAM size $M$, and are loading in 4 blocks (one for $\boldsymbol{Q},\boldsymbol{K},\boldsymbol{V}, \boldsymbol{O}$). I'm not sure why $B_r$ has the size that it does.

We then focus on a particular $\boldsymbol{K}_j,\boldsymbol{V}_j$ (this is out outer loop), and load these into SRAM.
After loading a $\boldsymbol{Q}_j,\boldsymbol{O}_j$ into SRAM (this is our inner loop), we compute a sub-block $\boldsymbol{S}_{ij}\in\mathbb{R}^{B_r\times B_c}$ :

![](_attachments/Screenshot%202022-11-06%20at%2013.54.33.png)

Now comes the clever part. In order to compute a softmax over $\boldsymbol{S}$ (row-wise), we need to keep track of:

1. The running max. We need it to be taken away from each element for stability reasons [^fn1].
2. The running total along the row. This allows us to compute the denominator of the softmax.

We compute both of these on-chip - see lines 10 & 11 of Algorithm 1. In doing so, we also compute a block of our $\boldsymbol{P}$ matrix:

![](_attachments/Screenshot%202022-11-06%20at%2013.55.08.png)

Finally, we **write out** our $\boldsymbol{O}$ block into DRAM (line 13):

![](_attachments/Screenshot%202022-11-06%20at%2013.55.31.png)

The next iteration of the inner loop computes the second row of $\boldsymbol{O}$:

![](_attachments/Screenshot%202022-11-06%20at%2013.56.33.png)

This repeats for all rows of $\boldsymbol{O}$. The next iteration of the outer loop updates each row again:

![](_attachments/Screenshot%202022-11-06%20at%2013.56.51.png)

And so forth. There are a couple points here that are useful to raise:

1. We load each $\boldsymbol{O}_i$ to and from DRAM several times; we do it for each iteration of the outer loop.
2. The speedup over Algorithm 0 is not coming from the $\boldsymbol{Q}\boldsymbol{K}^T$ operation, since this is tiled anyway. Instead, the benefit is derived from the way they're about to fuse the computation of $\boldsymbol{S} \to \boldsymbol{P} \to \boldsymbol{O}$. This is facilitated by the softmax trick. The fact that they don't save out $\boldsymbol{S}$ also saves memory writes.

Since $\boldsymbol{S}$ isn't stored for the backwards pass, they instead recompute $\boldsymbol{S}$ and $\boldsymbol{P}$ on the fly. This can be seen as a form of selective gradient checkpointing. But it's even cooler: gradient checkpointing usually involves a tradeoff between memory and compute; but this recomputation speeds up the backwards pass due to reduced DRAM accesses. So it's a win-win.

## I/O Analysis
I think it's quite nice to calculate the number of memory accesses required for standard attention implementations, and FlashAttention. 

> [!THEOREM 1]
> Let $N$ be sequence length, $d$ be the head dimension, and $M$ be the size of SRAM with $d \leq M \leq ND$. Standard attention (Algorithm 0) requires $O(Nd+N^2)$ DRAM accesses, while FlashAttention requires $O(N^2d^2M^{-1})$. 
> 

For typical values of $d$ (64-128) and $M$ (~ 100KB), $d^2$ is many times smaller than $M$. Thus, $d^2/M \ll 1$ so FlashAttention requires fewer DRAM accesses than Algorithm 0. 

> [!INFO]
> This implies that have a really big feature dim would wipe out the gains of FlashAttention (or this implementation, at least).

**Proof.**
We first consider Algorithm 0. Computing $\boldsymbol{S}=\boldsymbol{QK}^T$ requires reading $\boldsymbol{Q}, \boldsymbol{K}$ from memory, and writing out $\boldsymbol{S}$. If we assume we can load the entire input matrices into memory, then we have $O(Nd + N^2)$ accesses.
The rest of the algorithm is similar.

We now consider FlashAttention. 

> [!TODO]
> Finish this proof.


## Extensions: Masking and Dropout
Appendix B3 of the original paper gives a slightly more detailed version of FlashAttention. 
Given inputs $\boldsymbol{Q},\boldsymbol{K},\boldsymbol{V}\in\mathbb{R}^{N\times d}$, we wish to compute $\boldsymbol{O}\in\mathbb{R}^{N\times d}$:

![](_attachments/Screenshot%202022-11-03%20at%2018.37.47.png)

where $\tau\in\mathbb{R}$ is some softmax scaling (typically $\frac{1}{\sqrt{d}}$), $\textrm{MASK}$ is a masking function that sets the entries of the input to $-\inf$, and $\textrm{dropout}(x,p)$ is applied to $x$ element-wise.

The full algorithm is below. A key point is that they have to save the pseudo-random number generator state $\mathcal{R}$ for the backward pass[^fn2]:

![](_attachments/Screenshot%202022-11-03%20at%2018.40.23.png)

## Flash Attention in Triton Language
Flash Attention has been implemented in Triton language but in some senses is the inverse of the original implementation. Instead of loading $\boldsymbol{K}$, $\boldsymbol{V}$ in the outer loop, we load $\boldsymbol{Q}$, $\boldsymbol{O}$ and they remain in SRAM throughput all inner loop iterations. 

I've taken the forwards method defined in this implementation and extracted the key parts:

``` python
@triton.jit
def _fwd_kernel(
    Q, K, V, sm_scale,
    TMP, L, M,  
    Out,
    stride_qz, stride_qh, stride_qm, stride_qk,
    stride_kz, stride_kh, stride_kn, stride_kk,
    stride_vz, stride_vh, stride_vk, stride_vn,
    stride_oz, stride_oh, stride_om, stride_on,
    Z, H, N_CTX,
    BLOCK_M: tl.constexpr, BLOCK_DMODEL: tl.constexpr,
    BLOCK_N: tl.constexpr,
):
	# Define accumulator
	acc = tl.zeros([BLOCK_M, BLOCK_DMODEL], dtype=tl.float32)
...
	q = tl.load(q_ptrs) # Load Q block
...
	# loop over k, v and update accumulator
    for start_n in range(0, (start_m + 1) * BLOCK_M, BLOCK_N):
	    start_n = tl.multiple_of(start_n, BLOCK_N)
        # -- compute qk ----
        k = tl.load(k_ptrs + start_n * stride_kn)
        qk = tl.zeros([BLOCK_M, BLOCK_N], dtype=tl.float32)
        qk += tl.dot(q, k, trans_b=True)
        
		"""Skipped lines corresponding to softmax tricks"""

		# Compute P
        p = tl.exp(qk - m_ij[:, None])

		"""Skipped a bunch more lines"""

        # update acc
        v = tl.load(v_ptrs + start_n * stride_vk)
        p = p.to(tl.float16)
        acc += tl.dot(p, v)

	"""Skipped yet more lines"""
	
	# Store accumulator
	tl.store(out_ptrs, acc)
```

Let's look at a similar series of diagrams to what we had above. These help us understand exactly what's going on:

![](_attachments/Screenshot%202022-11-06%20at%2014.38.40.png)

![](_attachments/Screenshot%202022-11-06%20at%2014.38.54.png)

![](_attachments/Screenshot%202022-11-06%20at%2014.39.12.png)

> [!INFO]
> What is the advantage of this implementation, over that of the original paper? In Triton, outer loop iterations are easily parallelised. Each is executed by a single "program". In contrast, I don't think the original implementation is easily parallelised. If we were to parallelise over the outer loops, we would hit issues; each update to a chunk of $\boldsymbol{O}$ must be made sequentially due to our softmax trick. Else, we would hit a kind of "race condition".




[^fn1]: For any $x\geq 89$, $\exp(x)$ will overflow in FP32.
[^fn2]: Another place that this is done is for gradient checkpointing, with dropout. To recompute activations, we need to know $\mathcal{R}$ as it was at the beginning of the forwards pass.
