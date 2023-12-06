```toc
```

## Introduction
**Policy Gradient Methods** are very popular and widely used (as of 2015, at least). Loosely, these are methods that optimise the policy **directly** instead of working with value functions.

There are a variety of methods for doing this; they are all based upon using the policy **gradient**. We will look at: 

1. Finite Difference Policy Gradient
2. Monte-Carlo Policy Gradient
3. Actor-Critic Policy Gradient (this is the most useful).

Actor-critic methods combine the ideas from this class with the [last class](Lecture%206%20-%20Value%20Function%20Approximation.md). They work with both value functions and policies.

Last lecture, we approximated the value or action-value function using parameters $\theta$:

![](_attachments/Screenshot%202022-10-25%20at%2016.25.50.png)

The function approximator might be a linear combination of features, or neural network. A policy was generated directly from the value function.

This is only one approach to function approximation. Sometimes, a more natural approach is to approximate the policy **directly**:

![](_attachments/Screenshot%202022-10-25%20at%2016.27.21.png)

We control the parameters $\theta$, which affect the distribution from which we pick our actions. We will again focus on **model-free** RL.
We need to figure out how to improve the parameters, $\theta$. The main mechanism for this is **gradient descent**.

Some of the advantages & disadvantages of policy-based RL are as follows:

![](_attachments/Screenshot%202022-10-28%20at%2016.05.42.png)

The better convergence properties of policy-based RL are the main reasons seen in the literature. But perhaps the number 1 reason to use them is that they're effective in high-dimensional action spaces; in value-based RL, we always need to compute a *maximum*. This operation can be expensive, and is avoided by policy-based RL.

### Rock-Paper-Scissors Example
Why would we ever want to learn a stochastic policy? Surely we always want to maximise our reward, and this is necessarily deterministic?

We can disprove this with an example: in **rock-paper-scissors**, any deterministic strategy is very easily exploited. The only optimal policy is to play uniformly, at random. 

### Aliased Grid-world Example
A second example where non-deterministic policies are necessary is in Partially Observed Environments. Consider the diagram below:

![](_attachments/Screenshot%202022-10-28%20at%2016.11.48.png)

We will constrict our features to only consider partial information. Specifically, our features are of the form (for all N, E, S, W):

![](_attachments/Screenshot%202022-10-28%20at%2016.14.08.png)

What's the best we can do using value-based RL:

![](_attachments/Screenshot%202022-10-28%20at%2016.15.12.png)

or policy-based RL:

![](_attachments/Screenshot%202022-10-28%20at%2016.15.28.png)

Using these features, there is no way to distinguish between the two grey squares. As a result, a deterministic policy would *have* to pick the same action in these two states:

![](_attachments/Screenshot%202022-10-28%20at%2016.16.34.png)

I.e. an optimal deterministic policy will get stuck and **never** reach the money. 
**Value-based RL learns a near-deterministic policy (greedy or $\epsilon$-greedy)**, so it will traverse the corridor for a long time.

In contrast, an optimal **stochastic** policy will randomly move E or W in grey states:

![](_attachments/Screenshot%202022-10-28%20at%2016.17.59.png)

This can be described as:

![](_attachments/Screenshot%202022-10-28%20at%2016.18.45.png)

It will reach the goal state in a few steps, with high probability. This means that policy-based RL can learn the optimal stochastic policy.

The summary here is that **whenever state aliasing occurs (which can happen with a partially observed environment), as stochastic policy can do better than a deterministic policy**. 

> [!INFO]
> **State Aliasing** occurs when two or more distinct states are conflated in a model's representation space.


### Policy Search
What does it mean to optimise a policy? We need to know the objective that we're optimising against. There are a few different candidates.

In episodic environments, we can use the **start value**:

![](_attachments/Screenshot%202022-10-28%20at%2016.24.42.png)

This basically says: if we always start in **start state** $s_1$, what's the total reward we will get? This only works when we have the notion of a start state (for example, when playing chess as White).

In continuing environments, we can use the **average value**:

![](_attachments/Screenshot%202022-10-28%20at%2016.25.07.png)

Or the **average reward per time step**:

![](_attachments/Screenshot%202022-10-28%20at%2016.25.29.png)

where $d^{\pi_\theta}$ is a **stationary distribution** of a Markov chain for $\pi_\theta$.

Exactly the same method applies to all of these methods. They pretty much follow the same gradient direction, so we don't really need to worry about which we are following. 
Policy based RL is an **optimisation problem**. These are typically split into **gradient-based** and **gradient-free** methods. Greater efficiency is possible using the former.
Hence, we will focus on **gradient descent**.

## Finite Difference Policy Gradient
In this lecture, we use **gradient ascent** on the policy $\pi_\theta(s,a)$. 
Finite difference does as follows:

![](_attachments/Screenshot%202022-10-28%20at%2016.32.47.png)

This uses $n$ evaluations to compute the policy gradient in $n$ dimensions. It is simple, noisy, inefficient - but sometimes effective. It also works for arbitrary policies.

## Monte-Carlo Policy Gradient

### Likelihood Ratios
Let's now consider ways to compute the policy gradient **analytically**. 
We assume our policy $\pi_\theta$ is differentiable whenever it is non-zero, and that we know the gradient $\nabla_\theta\pi_\theta(s,a)$. 

We use **likelihood ratios**, which exploit the following identity:

![](_attachments/Screenshot%202022-10-28%20at%2016.38.21.png)

The **score function** is defined as $\nabla_\theta \log\pi_\theta(s,a)$. This has strong relationships to Maximum Likelihood Estimation. Re-writing our gradient in this way allows us to take expectations easily.

Let's first understand this score function. We'll use a **linear softmax policy** as a running example. This is used for discrete actions. 
We want to have a smoothly parametrised policy that informs us how frequently to choose an action. It's an alternative to something like $\epsilon$-greedy. 

We form a linear combination of features $\phi(s,a)^T\theta$. And then say that the probability of an action is proportional to the exponentiated weight:

![](_attachments/Screenshot%202022-10-28%20at%2016.42.11.png)

The score function is then:

![](_attachments/Screenshot%202022-10-28%20at%2016.42.27.png)

This is a nice way to parametrise policy in a discrete domain. 

Another example of a policy is a **Gaussian policy**. These are natural in continuous action spaces. We parametrise the **mean** of a Gaussian with a linear combination of features $\mu(s)=\phi(s)^T\theta$. The variance $\sigma^2$ may be fixed, or can be parametrised.

The policy is a Gaussian, i.e. $a\sim\mathcal{N}(\mu(s),\sigma^2)$.
The score function is:

![](_attachments/Screenshot%202022-10-28%20at%2016.46.21.png)

### Policy Gradient Theorem
Let's now actually look at what the policy gradient is.
Consider a simple class of **one-step** MDPs. I.e. we start in a state $s\sim d(s)$. We then terminate after one time-step with reward $r=\mathcal{R}_{s,a}$.

We use likelihood ratios to compute the policy gradient:

![](_attachments/Screenshot%202022-10-28%20at%2016.53.25.png)

To generalise this to multi-step MDPs, all we need to do is replace the immediate reward $r$ with the long-term value $Q^\pi(s,a)$. The **policy gradient theorem** formalises this:

![](_attachments/Screenshot%202022-10-28%20at%2016.58.08.png)

Let's now consider the simplest way to use this to construct an algorithm. This is called the **Monte-Carlo Policy Gradient**. The idea is to update parameters by stochastic gradient ascent. We use the return $G_t$ as an unbiased sample of $Q^{\pi_\theta}(s_t,a_t)$. 

For each step, we then adjust our parameters accordingly:

![](_attachments/Screenshot%202022-10-31%20at%2020.59.11.png)

Note that $v_t$ should be $G_t$ here!

## Actor-Critic Policy Gradient
As in previous lectures, Monte-Carlo policy gradient has high variance.
Instead of using a return to estimate the action-value function, we can instead use a **critic**:

![](_attachments/Screenshot%202022-10-31%20at%2021.01.40.png)

This uses the ideas of last lecture (i.e. value function approximation). Actor-critic algorithms maintain **two** sets of parameters:

![](_attachments/Screenshot%202022-10-31%20at%2021.02.17.png)

Intuitively, the actor is the model that actually *does stuff*. It picks actions. The critic doesn't take decisions - it simply observes the actor, and evaluates its actions. 

Actor-critic methods follow an **approximate policy gradient**:

![](_attachments/Screenshot%202022-10-31%20at%2021.02.45.png)

Put simply: the actor moves in a direction suggested by the critic.

The critic is solving a familiar problem: policy evaluation. This was explored in the previous two lectures. We could use Monte-Carlo policy evaluation, or TD learning.

To make things concrete, consider this canonical example:

Suppose we use a linear value function approximator $Q_w(s,a)=\phi(s,a)^Tw$. The critic updates $w$ by linear TD(0). The actor updates $\theta$ by policy gradient methods:

![](_attachments/Screenshot%202022-10-31%20at%2021.08.07.png)

For each step, we adjust both the actor and critic parameters.

### Advantage Function Critic
How can we improve our algorithms? The first trick is to reduce variance using a **baseline**. The idea is to subtract some baseline function $B(s)$ from the policy gradient.
This can reduce variance, without changing the expectation (i.e. the direction of gradient ascent):

![](_attachments/Screenshot%202022-10-31%20at%2021.22.58.png)

A good baseline is the *state* value function $B(s)=V^{\pi_\theta}(s)$. 
We thus rewrite the policy gradient using the **advantage function**, $A^{\pi_\theta}(s,a)$:

![](_attachments/Screenshot%202022-10-31%20at%2021.24.19.png)

This basically tells us: "how much better than usual is it to take action $a$"? Intuitively, this is the correct quantity to use in our update.  

How do we estimate our advantage function? There are several ways to do this. Here, we suggest a couple.
First, the critic could estimate **both** $V^{\pi_\theta}(s)$ and $Q^{\pi_\theta}(s,a)$. This uses two function approximators and two parameter vectors, and updates **both** value functions.

A second approach uses the following logic:

![](_attachments/Screenshot%202022-10-31%20at%2021.29.51.png)

This approach only requires one set of critic parameters $v$.

### Compatible Function Approximation
There is a really important question in actor-critic algorithms: we have used a critic as a replacement for the "true" critic value; is our gradient still going to be correct? 

Approximating the policy gradient introduces bias. And a biased policy gradient may not find the right solution.
Luckily, if we choose value function approximation carefully then we can avoid introducing bias. In other words, we can still follow the **exact** policy gradient.

In order for this to happen, there are **two** conditions that must be satisfied:

![](_attachments/Screenshot%202022-10-31%20at%2021.20.17.png)

### Eligibility Traces
What about eligibility traces? A critic can estimate the value function $V_\theta(s)$ from many targets at different time scales. From the last lecture:

![](_attachments/Screenshot%202022-10-31%20at%2021.33.05.png)

We can plug in exactly the same idea to our actors:

![](_attachments/Screenshot%202022-10-31%20at%2021.33.48.png)

So we can do policy gradient with eligibility traces in a way that's directly analogous to previous lectures:

![](_attachments/Screenshot%202022-10-31%20at%2021.36.27.png)

To be clear: these are all different ways to approximate our original policy gradient theorem! 

## Summary of Policy Gradient Algorithms

![](_attachments/Screenshot%202022-10-31%20at%2021.45.55.png)

All of these are different variants of the same idea. Each leads to a stochastic gradient ascent algorithm. 

The critic uses **policy evaluation** (e.g. MC or TD learning) to estimate $Q^\pi(s,a)$$, $A^\pi(s,a)$, or $V^\pi(s)$.

## Summary of Value-Based and Policy-Based RL
In the lectures, there is a slide summarising value-based and policy-based RL in the intro. I thought it made more sense to put it at the end:

![](_attachments/Screenshot%202022-10-31%20at%2021.13.51.png)

We only need $\epsilon$-greedy methods when we don't represent it explicitly. This lecture considers methods in the centre or right regions.