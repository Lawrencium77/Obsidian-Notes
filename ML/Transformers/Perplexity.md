These notes are almost entirely based on [this](https://en.wikipedia.org/wiki/Perplexity) Wikipedia article. 

### Perplexity as Normalised Inverse Probability of the Test Set
Suppose we've an unknown probability distribution $p$, used to generate some training samples. Given a probability model $q$, we can evaluate $q$ by asking how well it predicts a test sample $x_1, x_2, \ldots, x_N$ drawn from $p$. The perplexity is defined as:

$$\textrm{Perplexity}(q) = \left( \prod_iq(x_i) \right)^{-\frac{1}{N}}$$

In other words, it's basically:

$$\textrm{Perplexity}=\frac{1}{\textrm{Normalised Probability of Test Data}}$$

which makes sense: we want to maximise the probability of the test data, and therefore minimise perplexity.

### Perplexity as Exponential of Cross Entropy
We can re-write the above equation as:

$$\textrm{Perplexity}(q)=\left( \prod_iq(x_i) \right)^{-\frac{1}{N}}=2^{-\frac{1}{N}\sum_i\log_2q(x_i)}$$

This exponent $-\frac{1}{N}\sum_i\log_2q(x_i)$ can be interpreted as a measure of the **cross-entropy** between $p$ and $q$:

$$\textrm{Perplexity}(q) = 2^{H(p,q)}$$

### Perplexity Per Word
The definition of perplexity as "*the normalised inverse probability of the test set*" can be rephrased as follows:

> [!IDEA]
> If the perplexity of a [language model](GPT-3.md) on some test data is $P$, then for each token in the test set, the model was as uncertain as if it were choosing uniformly from $P$ tokens.

This is quite a nice way to characterise the "confusion" the model exhibited over the entire test set.







