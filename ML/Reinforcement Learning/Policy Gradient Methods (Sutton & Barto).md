I just made a few notes on this topic to solidify what was covered in the David Silver [lecture](Dave%20Silver%20Course/Lecture%207%20-%20Policy%20Gradient%20Methods.md). I just describe the need for the policy gradient theorem, and what it is.

```toc
```

# Intro
This chapter considers methods that learn a **parameterised policy** that can select actions without considering a value function. A value function may still be used to *learn* that policy parameter, but it's not required for action selection.

We use $\theta$ to represent the policy parameters, and $w$ to represent the value function's parameters (when appropriate). We consider methods for learning the policy parameter based on the gradient of some scalar performance measure $J(\theta)$ w.r.t $\theta$. Since we seek to maximise performance, we use approximate gradient ascent in $J$.

All methods that follow this general approach we will call **policy gradient methods**, regardless of whether they also learn an approximate value function. Methods that learn approximations to both policy and value functions are often called **actor-critic** methods.

We'll first consider the episodic case, in which performance is defined as the value of the start state under some parameterised policy. We'll then consider the continuing case, in which performance is defined as the average reward rate.

# The Policy Gradient Theorem
Policy parameterisation has an important theoretical advantage of $\epsilon$-greedy action selection. With continuous policy parameterisation, the action probabilities change smoothly as a function of the learned parameter. Whereas in $\epsilon$-greedy selection the action probabilities may change dramatically for an arbitrarily small change in the estimated action values (if that change results in a different action having the maximal value).
Largely because of this, stronger convergence guarantees are available for policy-gradient methods than for action-value methods.

The episodic and continuing cases define the performance measure, $J(\theta)$, differently and thus have to be treated separately to some extent. In this section, we treat the episodic case. We simplify the notation by assuming that every episode starts in some particular state $s_0$. Then, in the episodic case, performance is defined as:

$$J(\theta)=v_{\pi_\theta}(s_0)$$
where $v_{\pi_\theta}$ is the true value function for $\pi_\theta$. 

It may seem challenging to alter $\theta$ in a way that ensures improvement. Performance depends on both the action selections and the distribution of states in which those selections are made - and both of these are affected by $\theta$. 
Given a state, the effect of $\theta$ on the actions, and therefore reward, can be computed in a relatively straightforward way. But the effect of the policy on the state distribution is a function of the environment and is typically unknown. How can we estimate the performance of the gradient with respect to the policy parameter when the gradient depends on the unknown effect of policy changes on the state distribution?

There is an excellent theoretical answer to this, in the form of the **policy gradient theorem**. It provides an analytic expression for the gradient of performance w.r.t the policy parameter that does **not** involve the derivative of the state distribution.
For the episodic case, it establishes that:

![|400](_attachments/Screenshot%202022-11-26%20at%2018.11.02.png)

The distribution $\mu$ is the on-policy distribution under $\pi$. Loosely speaking, this describes the fraction of time spent in each state. There is a proof of this on p.325 of Sutton & Barto.

# Policy Gradient for Continuing Problems
For continuing problems without episode boundaries we need to define performance in terms of the average rate of reward per time step:

![|400](_attachments/Screenshot%202022-11-26%20at%2018.29.05.png)

In this case, $\mu(s)$ is the steady-state distribution under $\pi$, i.e. a stationary distribution.
In the continuing case, the policy gradient theorem is identical to above, except the constant of proportionality is one:

$$\nabla J(\theta)=\sum_s\mu(s)\sum_aq_\pi(s,a)\nabla\pi(a|s,\theta)$$









