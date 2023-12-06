These notes give a high-level description of how the [TXL](TXL%20Paper.md) works. They also make a couple of points that are applicable to transformers in general.
```toc
```
#### Part I: Motivation
It is easiest to start by considering an autoregressive transformer such as [GPT-3](GPT-3.md). During evaluation, each new token must attend to every other previous token in the context. This looks something like:

![](_attachments/Screenshot%202022-09-28%20at%2016.50.30.png) 

The computational complexity required to generate $N$ tokens is therefore $O(\sum_{k=1}^Nk)=O(\frac{N(N-1)}{2})=O(N^2)$.

In practice, we implement a finite window size due to constraints on compute and memory. The context used to generate a token can be no bigger than the window size.

In the original TXL paper, they include this diagram to describe the evaluation phase of a vanilla transformer:

![](_attachments/Screenshot%202022-09-28%20at%2016.55.07.png)

I originally found this confusing - it seems that each token is produced using previous context, so there should be some kind of recursion relation involved. It turns out this is wrong: the authors are suggesting that representations from previous tokens are recomputed *every* time we create a new token.

This is obviously extremely expensive, limits context, and causes context fragmentation. This is dumb. In fact it is so dumb that I'm skeptical that anyone *really* implements transformers like this. But it illusrates the point.

#### Part II: TXLs During Eval Mode
There is an obvious fix here: just cache the intermediate tokens. This leads to a recursion relation that signficantly increases the context for a single token:

![](_attachments/Screenshot%202022-09-28%20at%2017.03.46.png)

This is what they're getting at in the eval diagram for a TXL in the original paper:

![](_attachments/Screenshot%202022-09-28%20at%2017.04.22.png)

The *effective context* for a token is $O(ND)$, where $N$ is the layer number, and $D$ is the context length. This also drastically increases inference speeds, since we cache $K/V$ vectors in the context to avoid recomputation.

#### Part III: Training
Vanilla transformers train by just splitting a file into chunks, and doing the next-token prediction task for individual chunks.

But for the TXL, we train by attending to the previous context once given a new window:

![](_attachments/Screenshot%202022-09-28%20at%2017.13.08.png)

There are a few points to note here:
1. We cache the *intermediate representations* of every layer in the context, and use our new $W_K,W_V$ matrices to get the $k/v$ vectors for the context. This is in contrast to eval mode, where we just cache the $k/v$ vectors themselves.
2. There is an *approximation* here. The IRs for the context were computed using weights that have since been updated (according to the rules of gradient descent).
3. Backprop: we consider the IRs for the context as leaf nodes. In other words, we don't differentiate thriough them.

There is a diagram in the original TXL paper that is pretty much the same as above:

![](_attachments/Screenshot%202022-09-28%20at%2017.16.16.png)

#### Part IV: TXL vs Invariant Transformer (IT)
In our code (as of 09/22), we use an Invariant Transformer (IT) instead of a TXL. What's the difference?

First, TXLs use different positional embeddings to the IT (which uses [Rotary Embeddings](Rotary%20Embeddings.md)).

Second, the IT uses a fixed context length for *every token*. In contrast, the TXL just gets every token in a window to attend to every preceeding token in the window & context. I.e:

![](_attachments/Screenshot%202022-09-28%20at%2017.21.17.png)

The main advantage is that the output of the IT is the same *regardless of window size*.

Third, ITs use a "consistent softmax". I'm not sure what this means.

#### Part V: Parallelising Attention, and Masks

^5151d2

This part isn't specific to TXLs; the parallel nature of the TXL is of course true of all transformers.

But I thought it worth reiterating that the attention mechanism can be parallelised during training *really well*. We can calculate the pairwise scores for each $q \cdot k$ in parallel with some simple matmuls (which GPUs are really good at):

![](_attachments/Screenshot%202022-09-28%20at%2017.25.34.png)

We then take a softmax, and do another operation with the values:

![](_attachments/Screenshot%202022-09-28%20at%2017.25.43.png)

The masking of future context is easy enough: we stick some $- \infty$ values in the softmax. In other words, we do $(QK^T)M$, where $M$ is a lower/upper triangular matrix.

In practice, the masking matrix won't be square for a TXL/IT. It will be rectangular in shape, since the key dimension will be longer than the query dimension. 
Another subtlety is how the mask for an IT differs to that of a TXL. The IT will additionally mask our keys that are too far in the past. For a square matrix, this would look a bit like:

![](_attachments/Screenshot%202022-09-28%20at%2017.30.03.png)

But take this diagram with a pinch of salt - it's definitely not the right shape for an IT!

#### Part VI: Decode
This is a last point I wanted to make. It applies neither to TXLs nor ITs, but is true for autoregressive generative transformers like GPT-n.

The parallelisable nature of training does not hold during evaluation; in order to predict token $n$, we must know token $n-1,n-2,...,1$. And we can only know these if we have run decode in order. 

In other words, we have to make each prediciton sequentially.

