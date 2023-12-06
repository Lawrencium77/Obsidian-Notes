Speculative sampling is an algorithm for accelerated transformer decoding. This idea was developed by [Deepmind](https://arxiv.org/pdf/2302.01318.pdf) and [Google](https://arxiv.org/pdf/2211.17192.pdf) independently of one another, although their papers were published at very similar times.

I originally saw it mentioned in [this](https://twitter.com/karpathy/status/1697318534555336961) Andrej Karpathy tweet. These notes were largely written using [this](https://jaykmody.com/blog/speculative-sampling/) blog.

## Autoregressive Decoding
The standard way of generating text from an LM is with an **autoregressive** decode. 

![](_attachments/Screenshot%202023-09-02%20at%2015.15.24.png)

The time complexity of this algorithm is:

$$O(N\cdot t_{\textrm{model}})$$

where $t_{\textrm{model}}$ is the time for a single forward pass.

This is **memory-bandwidth bound**; most of the work is done in transferring [transformer](../Google%20Transformer%20Blog.md) weights from [DRAM](../../../Computing/GPUs/GPUs.md) into SRAM. This means that generating a single token from an LLM takes about as much time as generating a single token for a batch of size $K>1$, for a larger $K$ than one might think. See [this](https://finbarr.ca/how-is-llama-cpp-possible/) Finbarr article for some maths behind this.

## Speculative Sampling
In **speculative sampling**, two models are used:

1. A smaller, faster **draft** model (Deepmind's 7B Chinchilla model)
2. A larger, slower **target** model (Deepmind's 70B Chinchilla model)

The idea is that the draft model *speculates* what the output will be, $K$ steps into the future. The target model then determines how many of those tokens to *accept*. Here's an outline:

> 1. The draft model decodes $K$ tokens in the regular autoregressive fashion.
> 2. In parallel, compute the **target model** logits for each element of the draft output sequence.
> 3. Compare the target and draft logits to determine how many of the $K$ tokens we want to keep, based on some **rejection criteria**. If a token is rejected, we **resample** it using a combination of the two distributions, and don't consider any more of the $K$ tokens.
> 4. If all $K$ tokens are accepted, we sample an additional final token from the target model.

Overall, this means between $1$ and $K+1$ tokens are decoded at each iteration: if no tokens are accepted, we resample meaning that at least $1$ token is accepted; if all $K$ tokens are accepted, we can also sample a final token from the target model probability distribution, giving a total of $K+1$ tokens decoded. 

### In Detail
The full algorithm is below:

![](_attachments/Screenshot%202023-09-02%20at%2015.41.41.png)

Let's think some more about their rejection criterion. Given a sequence of tokens $x_1, \dots, x_n$ and $K$ draft tokens $\tilde{x}_{n+1}, \ldots, \tilde{x}_{n+K}$, they accept $\tilde{x}_{n+1}$ with probability:

$$\min \left(1, \frac{q\left(\tilde{x}_{n+1} \mid x_1, \ldots, x_n\right)}{p\left(\tilde{x}_{n+1} \mid x_1, \ldots, x_n\right)}\right)$$
They're basically just taking the ratio of the target and draft logits. 

If $\tilde{x}_{n+1}$ is rejected, they resample $x_{n+1}$ from the following distribution:

$$
x_{n+1} \sim\left(q\left(x \mid x_1, \ldots, x_n\right)-p\left(x \mid x_1, \ldots, x_n\right)\right)_{+}
$$

where $(.)_{+}$ denotes

$$
(f(x))_{+}=\frac{\max (0, f(x))}{\sum_x \max (0, f(x))}
$$

In other words, they are doing a **truncation** and **normalisation**. The reason this is necessary is that $x_{n+1} \sim\left(q\left(x \mid x_1, \ldots, x_n\right)-p\left(x \mid x_1, \ldots, x_n\right)\right)$ may not be a valid probability distribution. By using the $(.)+$ operation, they are making sure that:

1. There are no negative probabilities.
2. The resulting PD sums to 1.

In other words, this guarantees they're sampling from a valid probability distribution. Overall, this **modified rejection sampling** can be shown to be mathematically equivalent to doing an autoregressive decode with the target model.

The time complexity of this algorithm is:

$$O\left(\frac{N}{r(K+1)} \cdot\left(t_{\text {draft }} K+t_{\text {target }}\right)\right)$$

where $N=T-K$ is the number of tokens we wish to decode. The first term equals the number of iterations in the `while` loop; the second is the time complexity of each iteration.

### An Example
Consider the phrase:

> [!QUOTE]
> The apple doesn't fall far from the tree.

Given just the first part - "The apple doesn't fall" - we can use speculative decoding with $K=4$:

1. The draft model speculates the output to be "far from the tree".
2. The target model looks at those tokens, and decides to accept them all, and also sample a final token (i.e. maybe it samples a full-stop ".")

As such, in a single iteration, we were able to decode 5 tokens instead of just a single token. However, this may not always be the case, consider instead the input "Not all heroes":

1. The draft model speculates the output to be "wear capes and hats".
2. The target model looks at those tokens, but decides to only accepts the first two "wear capes" and discard the rest.

In this case, only 2 tokens were accepted.

The key point is that:

>[!KEY IDEA]
>As long as the draft model is sufficiently faster than the target model **while also** maintaining a high enough **acceptance rate**, then speculative sampling should yield a speedup.

### Intuition

The intuition behind speculative sampling is that certain strings of tokens (common phrases, punctuation, etc) are easy to predict, so a smaller but faster draft model should be able to quickly predict these instead of having our slower target model doing all the work.

### Benefits
An important property of speculative sampling is that it's **mathematically equivalent** to sampling from the target model. This is due to the way the rejection criteria and resampling method are defined.  ^8d24f9

In addition, speculative sampling requires no changes to the model architecture, or training. It is orthogonal to flash attention and quantization. 

