I thought this paper was pretty neat, so I made some summary notes on it. Kinda breaks my normal Lindy Effect rule... but who cares!

My main sources of info are the [original paper](https://arxiv.org/pdf/2212.03827.pdf) and its [LessWrong post](https://www.lesswrong.com/posts/L4anhrxjv8j2yRKKp/how-discovering-latent-knowledge-in-language-models-without). This [Twitter thread](https://twitter.com/CollinBurns4/status/1600892261633785856) is also good.

```toc
```

## Motivation
One risk of LMs is that they don't always tell the truth. Common training objectives can cause models to learn representations *related* to the truth, since truth is a useful feature for many tasks. But they can also cause LMs to output text that is false. For instance, training a model to imitate human-generated text may encourage it to output common misconceptions.

This issue stems from the misalignment between a training objective and the truth. 

## The Method: High-Level Summary
Instead of trying to explicitly specify truth, we search for implicit, internal "knowledge" learned by a model. We do so by leveraging some logical consistency properties that truth must satisfy, which are unlikely to be satisfied by other features.

We call this approach **Contrast-Consistent Search (CCS)**. 

It learns a projection of hidden states (just a one-layer network) that is consistent across negations, i.e:

$$p(\textrm{True}) + p(\textrm{Not True}) = 1$$

Here's a diagram:

![](_attachments/Screenshot%202023-02-20%20at%2019.22.33.png)

Description:

> We begin with a set of yes-no questions, $\{q_i\}_i$
> For each question, let $x_i^+$ and $x_i^-$ be the natural language statements where we answer the question $q_i$ as "Yes" and "No" respectively. Answering $q_i$ then amounts to determining which of $x_i^+$ and $x_i^-$ is true.
> Given the inputs $x_i^+$ and $x_i^-$, we map the internal activations of the model to a probability of being true. We do so with a learned mapping from the hidden states to a number between 0 and 1. The mapping must be both logically consistent, and confident (to avoid it just mapping to 0.5).

Let's be clear: this model is purely unsupervised - only leveraging the consistency of truth - but still learns to accurately answer questions.

## The Method: More Detail
Let $\phi(x)\in\mathbb{R}^d$ denote some feature representation on a natural language input $x$, such as the hidden state of a transformer. The goal is to answer the questions $q_1,\dots,q_n$ only given access to $\phi(\cdot)$

The input to CSS is a set of Yes-No questions $q_1,\dots,q_n$ and access to a pre-trained model's representations, $\phi(\cdot)$. The output of CSS is a lightweight probe on top of $\phi(\cdot)$ that can answer new questions. Crucially, CSS doesn't modify the weights of the pre-trained model, and does not use labels.

### Feature Extraction and Normalisation
Given a pair $(x_i^+, x_i^-)$, CCS first computes the representations $(\phi(x_i^+), \phi(x_i^-))$.
Intuitively, there are two key differences between $\phi(x_i^+)$ and $\phi(x_i^-)$: 

1. $\phi(x_i^+)$ ends with "Yes", while $\phi(x_i^-)$ ends with "No"
2. One is true, while the other is false

We want to find (2) rather than (1). So we first remove the effect of (1) by normalizing $\{\phi(x_i^+)\}$ and $\{\phi(x_i^+)\}$ independently. Specifically, we do:

![|500](_attachments/Screenshot%202023-02-20%20at%2019.43.56.png)

where $(\mu^+, \sigma^+)$ and $(\mu^-, \sigma^-)$ are the means and standard deviations of $\{\phi(x_i^+)\}_{i=1}^n$ and $\{\phi(x_i^-)\}_{i=1}^n$  respectively.
This normalisation ensures they no longer form two separate clusters.

### Mapping Activations to Probabilities
Next, we learn a probe $p_\theta$ that maps a normalised hidden state $\tilde{\phi}(x))$ to a number between 0 and 1 representing the probability that $x$ is true. We just use a linear projection followed by a sigmoid:

$$p_{\theta,b}(\tilde{\phi})=\sigma(\theta^T\tilde{\phi}+b)$$

For simplicity, we shall omit the $b$ subscript in $p$.
We extract the hidden states corresponding to the last token in the last layer of the LM.

### Training Objective 
First, we use the fact that a statement and its negation should have probabilities that add up to 1. This motivates the consistency loss:

$$L_{\textrm{consistency}}:=[p_\theta(x_i^+)-(1-p_\theta(x_i^-))]^2$$

However, this has a degenerate solution: $p(x^+)=p(x^-)=0.5$. To avoid this problem, we also encourage the model to be confident:

$$L_{\textrm{confidence}}:=\min{p_\theta(x_i^+),p_\theta(x_i^-)}$$

We can interpret $L_\textrm{confidence}$ as imposing a second consistency property on the probabilities: the law of the excluded middle (every statement must be true or false). The final unsupervised loss is the sum of these two losses, averaged across all contrast pairs:

$$L_{CCS}(\theta, b)=\frac{1}{n}\sum_{i=1}^nL_{\textrm{consistency}}(\theta,b;q_i)+L_{\textrm{confidence}}(\theta,b;q_i)$$

### Inference
It is possible that $p(x_i^+)$ and $1-p(x_i^-)$ are not exactly equal. To make a prediction on an example $x_i$ after training, we therefore take the average of these:

$$\tilde{p}(q_i)=\frac{1}{2}(p(x_i^+)+(1-p(x_i^-)))$$

We then predict whether the answer to $q_i$ is "Yes" based on whether $\tilde{p}(q_i)$ is greater than 0.5.

#### How Zero Shot Works
Zero-shot isn't part of their technique. But we use it as a baseline to compare against. 
The easiest way to do zero-shot would be to measure the logits across "Yes" and "No" output labels, $l_+$ and $l_-$, given a prompt $q_i$.

One complication is that zero-shout outputs sometimes suffer from "miscalibration", in which models are biased towards predicting specific answers. Calibrating the outputs to be uniform over different answers can mitigate this. Instead of classifying an example as positive if $l_+\gt l_-$ , we classify it as positive if $l_+\gt l_- + \gamma$, where $\gamma$ is selected such that predictions are balanced.




















