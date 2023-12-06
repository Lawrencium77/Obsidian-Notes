
```toc
```

## The Fisher Information Matrix
#### Intuition and the Score 
The Fisher Information Matrix (FIM) is a way of measuring the amount of information that an observed random variable, $X$[^1], carries about an unknown parameter $\theta$ that specifies its probability distribution.

A key idea is that if $p$ is sharply peaked with respect to $\theta$, then it is easy to infer the "correct" value of $\theta$[^2]. Equivalently, $X$ provides a lot of *information* about $\theta$.

I think that the above paragraph is a fairly obvious idea but is worth paying attention to. In particular, what does it mean for our PD to be "sharply peaked with respect to $\theta$"?

To gain some insight, we will consider a $p(X|\theta)$ that is a Gaussian, $\mathcal{N}(X|\theta, \sigma^2)$ [^3]. It can be justified with Bayes' theorem that a graph of $p(X|\theta)$ vs $\theta$ is also Gaussian.

We can now draw a graph of $log(p(X|\theta))$ vs $\theta$ for a given observation $X_i$ that is sampled from $p(X|\theta)$. It is important to understand that each $X_i$ is sampled from the underlying *true* PD, yet implies a form of $log(p(X|\theta))$ with $\theta = X_i$.

![](_attachments/Screenshot%202022-06-11%20at%2017.19.26.png)

At $\theta = \theta_{true}$, it is clear that the mean gradient across all of the curves is $0$. To put it formally, we define the **score** as $s(\theta) = \frac{\partial}{\partial\theta}(log(p(X|\theta)))$. For each curve, the score is telling you how you should update your estimate of $\theta$. When $\theta = \theta_{true}$, as you sample $X$ from the true PD, it's expected that this is $0$.

We can show this mathematically, too:

![](_attachments/Screenshot%202022-06-11%20at%2017.26.12.png)

#### The FIM as the Covariance of the Score
Let's plot the probability of $s(\theta)$ for the previous graph, when evaluated at $\theta = \theta_{true}$. We will again have a Gaussian:  

![](_attachments/Screenshot%202022-06-11%20at%2017.32.11.png)

Now, we allow ourselves to visualise what might happen if the graph of $log(p(X|\theta)$ becomes more or less sharply peaked in $\theta$-space. We won't justify it in detail, but it's kinda obvious that narrowing $p(X|\theta)$ will cause a greater spread of gradients measured at $\theta=\theta_{true}$, and vice versa:

![](_attachments/Screenshot%202022-06-11%20at%2017.35.30.png)

We can now motivate that the <u>variance of scores</u> tells us about the width of $log(p(X|\theta)$ in  $\theta$-space. Indeed, this is the definition of the FIM: 

![](_attachments/Screenshot%202022-06-11%20at%2017.39.36.png)

This can also be written in the following, commonly seen matrix form:

![](_attachments/Screenshot%202022-06-11%20at%2017.40.30.png)

If $log(p(X|\theta)$ is twice differentiable with respect to $\theta$, then the relation to curvature becomes even clearer. Indeed, the Fisher is the expected value of the Hessian of the log likelihood, when evaluated at $\theta_{true}$:

![](_attachments/Screenshot%202022-06-11%20at%2017.41.50.png)

#### Link with KL Divergence
By taking the definition of KL divergence and differentiating twice with respect to $\theta$, we can show that the <u>Fisher is the Hessian of the KL Divergence between $p(x|\theta)$ and $p(x|\theta')$, when evaluated at $\theta = \theta'$</u>:

![](_attachments/Screenshot%202022-06-11%20at%2018.27.48.png)

What's the intuition for this? Turns out it's fairly simple. The curvature of $log(p(X|\theta)$ determines how much it will "overlap" with another version of itself, as its parameters are slightly changed:

![](_attachments/Screenshot%202022-06-11%20at%2018.29.41.png)

We can draw a graph of how KL Divergence changes as the mean of a Gaussian distribution is altered. The FIM determines the curvature about the minimum:

![](_attachments/Screenshot%202022-06-11%20at%2019.41.59.png)

Another way of phrasing this is that the FIM defines the local curvature in distribution space, for which KL divergence is the metric. This comes in handy when discussing Natural Gradient Descent...

## Natural Gradient Descent (NGD)
We can motivate NGD by first considering vanilla gradient descent. In GD, we try to minimise some loss $L(\theta) = -log(p(X|\theta))$ within a given neighbourhood of $\theta$, as measured by the Euclidean distance[^4].

But why do we do this? There is no reason to believe that this approach is optimal. Indeed, instead of minimising the loss for a given $\delta \theta$, it might make more sense to control for the change in $log(p(X|\theta))$. This is ultimately what we care about!

Put another way: the purpose of NGD is to make sure we move through distribution space at constant speed. This makes sense especially for neural nets, where small changes in $\theta$ can lead to arbitrary changes in the PD that the model actually implements.

This is where the FIM comes in. Assuming we are at $\theta_{true}$, we can show that the 2nd-order Taylor expansion of KL divergence as we change $\theta$ is a function of the FIM:

![](_attachments/Screenshot%202022-06-11%20at%2018.38.49.png)

Simplifying further:

![](_attachments/Screenshot%202022-06-11%20at%2018.39.06.png)

This makes perfect sense when we look at the last picture in the previous section; we are at a minimum, which we can approximate with a quadratic. Hence the second order Taylor expansion is purely a function of the curvature, which is given by the FIM!

Now, we would like to know what the update vector $d$ that minimizes the loss function $L(θ)$ in distribution space. This is done with the following minimisation: 

![](_attachments/Screenshot%202022-06-11%20at%2018.43.44.png)

The rest of the mathematical details are not important (nor are they particularly complex). Suffice to say that the new parameter updates are a function of the inverse of the FIM. This is called the **Natural Gradient:**

![](_attachments/Screenshot%202022-06-11%20at%2018.45.16.png)

And that's it! 

As a side note, NGD could be thought of as an antidote to **catastrophic forgetting**, since it guarantees that the function your neural net implements doesn't change too much during training. This might also be related to why NGD is seen to work particularly well with **model averaging**.

[^1]: $X$ can represent a single sample drawn from a single distribution or can represent a collection of samples drawn from a collection of distributions.
[^2]: There is an implicit assumption here that the "true" underlying PD can be described by $p(x|\theta)$ for some "true" value of $\theta$.
[^3]: Excuse the abuse of notation here. In one instance, $\theta$ refers to all the parameters of the PD. When specifying a Gaussian, we use it to specify the mean.
[^4]: I'm not 100% sure how this interpretation works, since the "neighbourhood" within which we are constrained is a function of the norm of the gradient, which is variable throughout training. But it'll do for now.