

```toc
```
## Introduction
[Last Lecture](Lecture%203%20-%20Planning%20by%20Dynamic%20Programming.md), we covered planning by **Dynamic Programming**. This can be used to solve (i.e. find the optimal behaviour for) a *known* MDP. This lecture aims to estimate the value function of an *unknown* MDP. [Next lecture](Lecture%205%20-%20Model-Free%20Control.md), we will look at Model-Free control, which aims to *optimise* the value function of an unknown MDP.

## Monte-Carlo Learning (MC)
MC methods learn directly from episodes of experience. It is **model-free**: we've no knowledge of MDP transitions/rewards.

**MC uses the simplest possible idea: value = mean return.**

It learns from *complete* episodes; it only works for episodic MDPs. In other words, we have to terminate at some point. 

Let's go into a bit more detail. Our goal is to learn $v_\pi$ from episodes of experience under policy $\pi$. This experience is defined by the history:

![](_attachments/Screenshot%202022-10-07%20at%2016.08.29.png)

Recalling that the *return* is the total discounted reward:

![](_attachments/Screenshot%202022-10-07%20at%2016.09.21.png)

and that the value function is the expected return:

![](_attachments/Screenshot%202022-10-07%20at%2016.09.44.png)

MC policy evaluation uses the *empirical mean* return instead of an *expected* return. The only issue here is *how* to calculate the mean. There are two different ways to do this:

1) **First-Visit MC Policy Evaluation**
To evaluate state $s$ we do the following *for each episode*: the first time step $t$ that state $s$ is visited in an episode, we increment a counter $N(s) \leftarrow N(s) + 1$,  and the total return $S(s) \leftarrow S(s) + G_t$. 
The value is estimated by the mean return: $V(s)=S(s)/N(s)$.
By the law of large numbers, $V(s)\to v_\pi(s)$ as $N(s)\to \infty$.

To be clear: this counter is operating over a set of episodes.

2) **Every-Visit MC Policy Evaluation**
To evaluate state $s$ we do the following *for each episode*: **every** time step $t$ that state $s$ is visited in an episode, we increment a counter $N(s) \leftarrow N(s) + 1$,  and the total return $S(s) \leftarrow S(s) + G_t$. 
The value is estimated by the mean return: $V(s)=S(s)/N(s)$.
By the law of large numbers, $V(s)\to v_\pi(s)$ as $N(s)\to \infty$.

To be clear: for one episode, you can increment the counter multiple times.

### Blackjack Example
Consider 200 states. Each state has three variables:
* Current sum of our two cards (only consider 12-21; if the sum is 11 or less then you will just always ask for another card).
* Card being shown by the dealer (ace-10) 
* Whether we have an ace in our hand (it can take value 1 or 11).

Two actions:
**Stick** (stop receiving cards)
**Twist** (take another card)

Reward for stick:
$+1$ if sum of cards > sum of dealer cards
$0$ if sum of cards $=$ dealer cards
$-1$ if sum of cards $\lt$ dealer cards

Reward for twist:
$-1$ if sum of cards $\gt$ 21
$0$ otherwise

Transitions: we automatically twist if sum of cards $\lt$ 12.

To apply policy evaluation here, we consider the policy: stick if sum of cards $\gt=$ 20, otherwise twist. Running lots of episodes gives us a good estimate of the value function:

![](_attachments/Screenshot%202022-10-07%20at%2016.33.58.png)

We need about 500k episodes to understand the shape of a value function in the case that we do have an ace.

### Incremental Monte Carlo
First, we define the **incremental mean**. The mean $\mu_1,\mu_2$ of a sequence $x_1,x_2$ can be computed incrementally:

![](_attachments/Screenshot%202022-10-07%20at%2016.40.21.png)

This is just pointing out that we can compute the mean in an online fashion.
Let's do this with the MC algorithm. We update $V(s)$ incrementally after each episode $S_1, A_1, R_2, \dots, S_T$. For each state $S_t$ with return $G_t$

![](_attachments/Screenshot%202022-10-07%20at%2016.43.01.png)

In non-stationary problems (where your environment is changing) it can be useful to track an exponentially moving average. This means that we would "forget" old episodes:

![](_attachments/Screenshot%202022-10-07%20at%2016.43.40.png)

## Temporal-Difference Learning (TD)
TD methods learn directly from episodes of experience. It applies to the non-episodic case too.
It's model free: we don't need a knowledge of MDP transitions/rewards.
It learns from **incomplete episodes** - this is a difference to MC. Intuitively, we take an partial trajectory and then use an estimate of any future return we think we'll get. This idea is called **bootstrapping**: we substitute the remainder of a trajectory with our guess of what will happen from the current state.

Let's make this more concrete.

The goal is the same as for MC: to learn $v_\pi$ online from experience under policy $\pi$. Considering the simplest version, TD(0), we update $V(S_t)$ toward *estimated* return $R_{t+1}+\gamma V(S_{t+1})$ (this is decomposed just like in the Bellman Equation).

![](_attachments/Screenshot%202022-10-07%20at%2016.53.05.png)

A couple of bits of terminology:

![](_attachments/Screenshot%202022-10-07%20at%2016.53.18.png)

### Online vs Offline TD Learning
> [!INFO]
> This section is not in the original lecture. I added it myself.

I thought it would be useful to quickly explain the distinction between online and offline TD learning.
For **online** learning, the updates to our value function are computed *during* the episode, as soon as an increment is computed: $V_{t+1}(s) :=V_t(s) + \delta_t$
For **offline** learning, the increments are accumulated "on the side" and are not used to change value estimates until the end of the episode: $V_{t+1}(s) :=V_t(s) + \sum_{t=0}^{T-1}\delta_T(s)$.

### Driving Home Example
Why is this a good idea? Suppose we are driving a car. Another car comes towards us. It looks as if we're about to crash, but it swerves out of the way at the last second.
In MC, we would not update our value function to reflect that we nearly crashed. But in TD learning, we can account for the near-death experience!

Let's consider another:

![](_attachments/Screenshot%202022-10-07%20at%2016.56.11.png)

How would we update our value function based on these experiences? The diagram below really shows the difference between MC and TD:

![](_attachments/Screenshot%202022-10-07%20at%2016.57.54.png)

In MC, we are comparing our predictions to what *actually* happened. You have to wait until the episode is complete in order to update. 
In TD, we are comparing our predictions to *future* predictions. So we can immediately update our value function.

### Pros/Cons of MC vs TD
TD can learn *before* knowing the final outcome. It can learn online after every step. In contrast, MC must wait until the end of an episode before the return is known.

TD can learn *without* the final outcome. It can therefore learn from incomplete sequences; MC can learn from complete sequences. TD works in continuing (non-terminating) environments; but MC only works for episodic (terminating) environments.

### Bias/Variance Trade-Off
Another major difference between these two algorithms is the bias-variance trade-off they make.

In MC, the return $G_t=R_{t+1}+\gamma R_{t+2}+\dots$ is an unbiased estimate of $v_\pi(S_t)$. In TD, the "true target" is $R_{t+1}+\gamma v_\pi(S_{t+1})$. This is also an unbiased estimator.

However, the TD target that we *actually* use $R_{t+1}+\gamma V_\pi(S_{t+1})$ is a *biased* estimate of $v_\pi(S_t)$. But the TD target is much lower variance than the return (which is used by MC).
Intuitively, this is because the return $G_t$ depends on *many* actions, transitions and rewards. There is a lot of noise over many steps.
But for the TD target, we only incur noise over one step.

### Pros/Cons of MC vs TD Part II
MC has high variance, but zero bias:
* Good convergence properties
* Not sensitive to initial value
* Very simple to understand and use

TD has low variance, but some bias:
* Usually more efficient than MC
* TD(0) always converges to $v_\pi(s)$
* (but not always with function approximation)[^fn1]
* More sensitive to initial value

### Random Walk Example
Consider a random walk. There are two actions (left & right). We consider a uniform random policy. The left- and right-most states are terminal. The rewards a labeled on the diagram below:

![](_attachments/Screenshot%202022-10-08%20at%2010.33.24.png)

This is perhaps the simplest MDP. Applying TD(0): 

![](_attachments/Screenshot%202022-10-08%20at%2010.34.42.png)

We begin with an initialisation of everything at 0.5. After 1, 10, 100 episodes we can see that our value function converges on the true value function.

Comparing MC to TD:

![](_attachments/Screenshot%202022-10-08%20at%2010.37.01.png)

These graphs show how the two algorithms converge, with different step sizes. TD does better than MC. This shows the **effectiveness of bootstrapping**; by bootstrapping, you can learn more efficiently by avoiding the variance that occurs later on in the episode.

### Batch MC and TD
MC and TD both converge: $V(s) \to v_\pi(s)$ as experience $\to \infty$.

But what if we stop after a finite number of episodes? And instead just repeat this experience again and again. This is the **batch** case.

In other words, we have:

![](_attachments/Screenshot%202022-10-08%20at%2010.40.08.png)

where we repeatedly sample episode $k \in [1,K]$.

To get an intuition for if MC & TD converge, we consider a simple example: the **AB Example**.
Suppose we have two states A,B; no discounting; 8 episodes of experience.

![](_attachments/Screenshot%202022-10-08%20at%2010.42.03.png)

Each row is a different episode of experience. On the first row, you start in state A, get a reward of 0, transition to state B, get a reward of 0, and then terminate. And similar for the other rows.

What is $V(A), V(B)$. It's clear that $V(B)$ is  $\frac{6}{8}$. 

$V(A)$ could legitmately be estimated as $0$ or $\frac{6}{8}$. If we follow MC, we see only one episode of A, and our estimate is $0$.
For TD, we are **implicitly building an MDP**: 

![](_attachments/Screenshot%202022-10-08%20at%2010.46.20.png)

Using this, we can understand what MC and TD converge to. 
MC converges to the solution with the minimum mean-squared error; it's the best fit to the observed returns:

![](_attachments/Screenshot%202022-10-08%20at%2010.48.21.png)

TD(0) converges to the solution of the MDP that best explains the data:

![](_attachments/Screenshot%202022-10-08%20at%2010.48.55.png)

### Pros/Cons of MC vs TD Part III
TD exploits the Markov property. This means that it's usually more efficient in Markov environments.

MC does not exploit the Makov property. So it's usually more effective in non-Markov environments.

### Unified View
We can think of our updates like backups. You start in some state, and there is implicity some kind of look-ahead tree.

![](_attachments/Screenshot%202022-10-08%20at%2010.53.08.png)

MC samples one complete trajectory, as shown in red. You use that sample to update your estimate of $S_t$. But you also use it to update your estimate at all states visited during the episode.

In contrast, TD learning can be illustrated as follows:

![](_attachments/Screenshot%202022-10-08%20at%2010.54.37.png)

The backup is just over one step. We don't backup all the way to the end of an episode.

In dynamic programming, we also did a one-step look ahead. But we considered all possible paths over that one step:

![](_attachments/Screenshot%202022-10-08%20at%2010.55.57.png)

We could do this because we didn't take a sample; we knew our dynamics, so could compute an expectation.

Let's now try and classify these algorithms according to different properties.

First, there's **bootstrapping**: the idea that you don't use the real returns, but use your own value function as a target.
* MC doesn't bootstrap
* TD & DP both bootstrap

Second, there's **sampling**:
* MC and TD sample;
* DP doesn't sample

We can illusrate this as below:

![](_attachments/Screenshot%202022-10-08%20at%2010.58.44.png)

## TD$(\lambda)$
There is a spectrum of methods between shallow backups and deep backups (on the x-axis above). These algorithms are refered to as TD($\lambda$). 

In a simple sense, we let our TD target look $n$ steps into the future:

![](_attachments/Screenshot%202022-10-08%20at%2011.03.03.png)

In other words, instead of taking 1 step and then using our estimate of the value function where we end up, why not take 2 steps, or 3 steps, or $n$ steps?

Writing out what this means mathematically:

![](_attachments/Screenshot%202022-10-08%20at%2011.05.14.png)

The only thing that changes during our update of $V(S_t)$ is our estimate of $G_t$. 

The natural question here is: which $n$ is the best? Here's a study of a larger random walk:

![](_attachments/Screenshot%202022-10-08%20at%2011.07.24.png)

The distinction of on-line vs off-line TD is whether or not we update our value function during an episode, or at the end.

We see that as $n \to \infty$, we get quite a high variance. In between TD(0) and MC, we get a sweet-spot. 

But this is quite unsatisfactory, as it's so problem-dependent. 

### Averaging $n$-Step Returns

Can we design an algorithm that is robust across all $n$?
Yes. We can average $n$-step returns over different $n$. For instance, we can average the 2-step and 4-step returns:

![](_attachments/Screenshot%202022-10-08%20at%2011.13.31.png)

This combines information from 2 different time steps. We can generalise this to combine information from all time steps:

![](_attachments/Screenshot%202022-10-08%20at%2011.14.30.png)

This defines a quantity called the $\lambda$-**return**. $\lambda \in [0,1]$. The weighting function looks something like:

![](_attachments/Screenshot%202022-10-08%20at%2011.17.47.png)

The $(1-\lambda)$ prefactor is just for normalisation. The summation in our formula is from $1$ to $\infty$, meaning we are only considering complete episodes.

It can be useful to decompose our above formula for $G_t^{(\lambda)}$ into return pre- and post-termination:

$G_t^{(\lambda)}=(1-\lambda)\biggr[\sum_{n=1}^{T-t-1} \lambda^{n-1}G_t^{(n)}+\lambda^{T-t-1}G_f(1-\lambda)^{-1}\biggr]=(1-\lambda)\biggr[\sum_{n=1}^{T-t-1} \lambda^{n-1}G_t^{(n)}\biggr]+\lambda^{T-t-1}G_f$

where $G_f$ is defined as being the $\lambda$-reward for the terminal state. Note that the second term is derived from evaluating an infinite series, $\lambda^{T-t-1}G_f\sum_{k=0}^\infty \lambda^k$. I have written a more formal proof below:

![](_attachments/Screenshot%202022-10-12%20at%2017.37.48.png)

We are now in a good position to understand the limiting cases for the values of $\lambda$. From inspecting the above equation:

* $\lambda=1 \to$ Monte Carlo
* $\lambda=0 \to$ TD(0)

### Forwards vs Backward View of TD($\lambda$)
So far, we have seen a forward-view algorithm:

![](_attachments/Screenshot%202022-10-08%20at%2011.20.34.png)

Let's now come up with a backward view, which achieves this but with the nice properties we had for TD(0), where it could be updated online and use incomplete sequences.

To do this, we must first consider **eligibility traces**. Suppose we are a rat, and we see three bells, a light, and then get an electric shock. What caused the shock - the bell or the light?

There are two heuristics to consider here: a frequency heuristic (assign credit to most frequent states), and a recency heuristic (assign credit to most recent states).

Eligiblity traces combine both heuristics:

![](_attachments/Screenshot%202022-10-08%20at%2011.27.47.png)

This is relevant to us. Now when we see an error, we update our value function according to the eligbility trace. 

This allows us to construct a **backward view of TD($\lambda$):**

![](_attachments/Screenshot%202022-10-08%20at%2011.29.07.png)

Intuitively, we are "shouting" back $\delta_t$ over time, updating states that we have seen previously. But the strength of our voice decays with temporal distance by $\gamma \lambda$.

How does this relate to the algorithms we saw previously?

![](_attachments/Screenshot%202022-10-08%20at%2011.30.22.png)

At the other extreme:

![](_attachments/Screenshot%202022-10-08%20at%2011.31.03.png)

The reasoning for this isn't fully explained in these notes. Look at the [lecture slides](https://www.davidsilver.uk/wp-content/uploads/2020/03/MC-TD.pdf) for more info.

Backwards-view TD($\lambda$) relates to TD(0) and MC in exactly the way as forwards-view TD($\lambda$). This is not a surprise. When you do *offline* updates, the forwards and backwards of TD($\lambda$) are exactly equivalent:

![](_attachments/Screenshot%202022-10-08%20at%2011.32.01.png)

Again, this is proved in the lecture slides.

### Summary of Forward and Backward TD($\lambda$)
This table summarises the relationship between forward and backward TD($\lambda$). We see that in the offline case, they are equivalent. 
When doing online updates, the backward view isn't exactly equivalent to the forwards view mechanism.

![](_attachments/Screenshot%202022-10-08%20at%2011.33.15.png)













[^fn1]: We'll see what this means in later lectures