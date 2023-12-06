I was asked this in my [FAR AI](../Random/Applications/Application%20Process/FAR%20AI/FAR%20AI.md) interview, so I thought it best to make some concrete notes.

### Setup
Assume we've a statistical model, parameterised by parameter $\theta$, that gives rise to a probability distribution $P_\theta(x)=P(x \mid \theta)$.

An **estimator** aims to estimate $\theta$ based on some observed data. Formally, it can be any function of the data:

$$\theta_m=g(x^1,\dots,x^m)$$

### Bias
The **bias** of an estimator is defined as:

$$\textrm{bias}(\hat{\theta})=\mathbb{E}_x[\hat{\theta}]-\theta$$

where $\mathbb{E}_x$ denotes an expectation over $P(x|\theta)$. 
In other words, it's the difference between the estimator's expected value and the true value of $\theta$.

### Variance
The variance of an estimator is simply:

$$\textrm{var}({\hat{\theta}})=\mathbb{E}(({\hat{\theta}}-\mathbb{E}({\hat{\theta}})^2)$$

### MSE
To reflect both types of difference, one measure we can use is MSE:

$$\begin{aligned}
\operatorname{MSE}(\hat{\theta}) & =(\mathrm{E}[\hat{\theta}]-\theta)^2+\mathrm{E}\left[(\hat{\theta}-\mathrm{E}[\hat{\theta}])^2\right] \\
& =(\operatorname{Bias}(\hat{\theta}))^2+\operatorname{Var}(\hat{\theta})
\end{aligned}$$

### Bias-Variance Tradeoff
The relationship between bias and variance is tightly linked to the concepts of capacity, underfitting, and overfitting. When generalisation error is measured by MSE, then:

* Increasing capacity $\to$ increase variance, decrease bias.
* Decreasing capacity $\to$ decrease variance, increase bias.

This idea is illustrated below:

![](_attachments/Screenshot%202023-08-27%20at%2010.57.37.png)

More capable models often have higher variance because they overfit, thus capturing dataset **noise**.

### In Neural Networks
Clearly, a small network may have higher bias than a large network. But a large network may have higher variance.

There are plenty of other factors that affect bias-variance tradeoff in neural nets, including architecture, regularisation, batch normalisation, etc.



