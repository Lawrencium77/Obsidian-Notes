(These notes were mainly based on [this blog](https://d2l.ai/chapter_recurrent-modern/beam-search.html).)

Decoding strategies describe how to convert token probabilities into actual output. This is of particular relevance in sequence modelling. We will describe these approaches:

1. [Greedy Search](#Greedy%20Search)
2. [Exhaustive Search](#Exhaustive%20Search)
3. [Beam Search](#Beam%20Search)

However, there are [other modifications that can be made](Sampling%20Strategies.md). These include random, top-$k$, and nucleus sampling.

## Setup
Suppose we have a [transformer](../Attention%20is%20All%20You%20Need.md) that produces output over time steps $t$. Each output $y_t$ is conditioned upon the previous tokens $y_1,\dots,y_{t-1}$.
To quantify computational cost, denote our vocabulary size as $Y$ and our maximum number of output tokens as $T$.

The goal is to search for the most probable of the $O(Y^T)$ possible output sequences. Note that this slightly overestimates the number of distinct outputs because there are no subsequent tokens after the `<eos>` token occurs.

## Greedy Search
For **greedy search**, at any time step $t$ we simply select the token with the highest conditional probability. To be precise:

$$y_t=\textrm{argmax}_{y\in Y}P(y|y_1,\dots,y_{t-1})$$

Once our model outputs `<eos>`, the output sequence terminates.
The computational cost of a greedy search is $O(T)$.


## Exhaustive Search
Since the goal is to obtain the most likely *sequence*, we may consider using exhaustive search. This is simply a **breadth-first search**.
The computational cost would be $O(Y^T)$, which makes it intractable.

## Beam Search
We can view decoding strategies as lying on a spectrum. **Beam search** strikes a balance between [Greedy Search](#Greedy%20Search) and [Exhaustive Search](#Exhaustive%20Search).

The most straightforward version of beam search is characterised by a single hyperparameter: the **beam size**, $k$.

At time step 1, we select the $k$ most probable tokens. At time step 2, we use the $k$ candidate tokens to generate further sequences and drop all but the $k$ most probable. And so forth.

[Wikipedia's description](https://en.wikipedia.org/wiki/Beam_search#:~:text=In%20computer%20science%2C%20beam%20search,that%20reduces%20its%20memory%20requirements.) is quite nice:

> [!QUOTE]
> Beam search uses breadth-first search to build its search tree. At each level of the tree, it generates all successors of the states at the current level, sorting them in increasing order of heuristic cost. However, it only stores a predetermined number, $k$, of best states at each level (called the beam width). Only those states are expanded next.

Here's a diagram:

![](_attachments/Screenshot%202023-03-04%20at%2015.43.23.png)

To be clear: at all time steps, we maintain only $k$ candidate sequences. [Exhaustive Search](#Exhaustive%20Search) is equivalent to beam search with an infinite beam width.

The computational cost of beam search is $O(kT)$.