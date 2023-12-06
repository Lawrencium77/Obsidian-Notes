[Medusa](https://www.together.ai/blog/medusa) is similar to speculative decoding in that it creates draft sequences which are then processed in parallel by the target model. However, they point out three challenges associated with speculative decoding:

1. **Finding the ideal draft model**: this is easier said than done.
2. **System complexity**: Hosting two distinct models is difficult, especially in distributed settings.
3. **Sampling inefficiency**: importance sampling adds additional overhead on generation, especially at higher sampling temperatures.

## Overview

Instead of using a separate draft model, Medusa extends the original model to produce draft sequences. Specifically, it adds extra "heads" to the model. They also use a **tree-based attention mechanism** to get an extra boost in processing speeds. Finally, they also use a slightly different sampling scheme, designed to reduce overhead further.

The following diagram summarises the approach:

![](_attachments/Screenshot%202023-12-05%20at%2009.06.34.png)

The original model is frozen when training the Medusa heads. Each head generates predictions for some number, $k$, of tokens ahead. In much the same way as [Speculative Decoding](Speculative%20Decoding.md), these predictions are processed in parallel using the base model and reject/accept tokens accordingly. 

The key benefits of this approach over Speculative Decoding are:

* You're leveraging the more powerful base model.
* It's cheap to train the Medusa heads.
* You can generate the candidate sequence in parallel.

## Medusa Heads
These heads are simply a single feed-forward layer, with a residual connection.

## Tree Attention
The purpose of tree attention is to leverage the top-$N$ predictions made by each head, rather than simply the top-1. 

They first create a set of candidate sequences by taking the **Cartesian product** of the top predictions from each head[^fn1]. They then process multiple candidates in parallel, using a clever attention mask.

For example, suppose we consider the top 2 predictions from the first head, and the top 3 from the second:

![](_attachments/Screenshot%202023-12-05%20at%2009.17.04.png)

The results in a multi-level tree structure. Within the tree, we implement an autoregressive attention mask (indicated by the blue ticks). By doing so, and by setting positional indices for positional embeddings accordingly, we can process multiple candidates simultaneously.

It's important to note that we process these candidates in a similar way to [Speculative Decoding](Speculative%20Decoding.md); we "accept" tokens in the sequence as long as they match the predictions that *would* have been made by the target model. Indeed, the "ongest accepted candidate prefix" is used for the next decoding phase. This makes sense when inspecting the [first image](#Overview).




Analogy between [branch prediction](Control#^2b0cbb) and assisted generation.


[^fn1]: In other words, they consider all possible combinations of the outputs of each head.