

```toc
```

## Introduction
In some sense, the whole course has been leading to this lecture. We will find out how an agent can maximise its reward in an unknown environment. In subsequent lectures, we will look at how to scale up these methods.

[Last lecture](Lecture%204%20-%20Model-Free%20Prediction.md), we saw how to do model-free prediction using Monte-Carlo learning and Temporal Difference Learning. This lecture, we use these methods to do model-free control.

Here are some example problems that can be modelled as MDPs:
* Controlling an elevator
* Game of Go
* Protein Folding

For all of these problems, there is an underlying MDP. But either:
* MDP model is unknown, but experience can be sampled
* MDP model is known, but is too big to use, except by samples.

This means that we need to use model-free control.

## On-Policy Monte-Carlo Control
It is first important to understand the difference between **on-policy** and **off-policy** learning.

For **on-policy** learning:
* Learn "on the job"
* Learn about a policy $\pi$ from experience sampled from $\pi$

In other words, the actions you take determine the policy that you're trying to evaluate.

For **off-policy** learning:
* Learn by "looking over someone's shoulder"
* Learn about a policy $\pi$ from experience sampled from $\mu$

### Generalised Policy Iteration
Just a quick refresher on policy iteration.
We alternate between two different processes (Iterative Policy Evaluation, and Greedy Policy Improvement):

![](_attachments/Screenshot%202022-10-10%20at%2021.13.37.png)

We can generalise this process by varying which algorithms we use for a) policy evaluation, and b) Policy improvement. 

Let's start with the simplest case. We'll stick in Monte-Carlo policy evaluation instead of our dynamic programming. We run some trajectories, use these to estimate $V$, and then use greedy policy improvement:

![](_attachments/Screenshot%202022-10-10%20at%2021.16.04.png)

However, there are two problems with this:

* Exploration issue: if you act greedily all the time, you don't explore enough of your state space.
* Greedy policy improvement over $V(s)$ requires a *model* of the MDP:

![](_attachments/Screenshot%202022-10-10%20at%2021.19.16.png)

The whole point is that we are model free! If you work with the state-value function, you always need a model.
The alternative is to use $q_\pi$:

![](_attachments/Screenshot%202022-10-10%20at%2021.20.36.png)

Obviously, both $Q(s,a)$ and $V(s)$ encode information about the environment dynamics. But $Q(s,a)$ allows us to optimise $\pi(s)$ in a model-free way.

What would this algorithm look like? It's the same as before, but we begin with some $Q,\pi$:

![](_attachments/Screenshot%202022-10-10%20at%2021.24.38.png)

### Exploration
However, the exploration issue still remains. Since we are acting greedily, we might not encounter some states & actions. 
There is a distinction here between model-free control and dynamic programming. For DP, greedy policy improvement still guaranteed convergence to $v_*/\pi_*$. But for model-free methods, this isn't the case.
Intuitively, this is because at each stage of DP, we consider every state in our MDP. But when using MC, we only sample some subset of them. So it's possible to get "stuck" in a kind of local maximum.

Let's look at an example to make this a bit clearer. Suppose we have two actions available at each state: left door or right door.

![](_attachments/Screenshot%202022-10-10%20at%2021.28.17.png)

The comments above show what you would do if you were acting greedily. Your MC estimate for the value of picking the right door is soon $>0$. We could keep on opening the right door forever. But we don't really know the value of opening the left door.

How do we make sure that we explore sufficiently? There will be a [whole lecture](Lecture%209%20-%20Exploration%20and%20Exploitation.md) on this, but we will start with the simplest idea. This is **$\epsilon$-greedy exploration.**

The idea is that when deciding an action, we pick the greedy action with probability $1-\epsilon$. Else, we choose an action at random. $\epsilon$ is a parameter that satisfies $\epsilon\in [0,1]$.
This policy can be written as follows:

![](_attachments/Screenshot%202022-10-10%20at%2021.41.12.png)

One of the reasons that this is a nice idea is that it **guarantees we get a step of policy improvement**:

![](_attachments/Screenshot%202022-10-10%20at%2021.42.22.png)

The idea of this proof is similar to what we've seen before. We consider taking one step of our new policy $\pi'$, and then follow previous policy $\pi$. We can then use a "telescoping" argument similar to in the [Lecture 3 - Planning by Dynamic Programming](Lecture%203%20-%20Planning%20by%20Dynamic%20Programming.md).

Let's look at our new, improved policy iteration algorithm:

![](_attachments/Screenshot%202022-10-10%20at%2021.46.37.png)

To be clear: we now have some stochasticity in our policy.  This ensures some rate of exploration, and guarantees convergence to $q_*, \pi_*$.

Let's make this more efficient. We saw in our [DP](Lecture%204%20-%20Model-Free%20Prediction.md) lecture that it is not always necessary to go *fully* evaluate our policy before doing policy improvement. What does this look like in the context of Monte-Carlo?
We can take this to the extreme, and do policy improvement after *every* episode:

![](_attachments/Screenshot%202022-10-10%20at%2021.49.22.png)

In other words, we react greedily to our newest update of the value function. 

### GLIE
How can we guarantee that we find $q_*/\pi_*$?
To reach convergence, we need to do two things: first, there must be sufficient exploration. Second, once we reach close to $q_*$, we need to no longer be exploring because the optimal policy will not involve some random behaviour.

One idea for balancing these two factors is **Greedy in the Limit with Infinite Exploration (GLIE)**. The idea is to come up with a schedule for exploration such that two conditions are met:

![](_attachments/Screenshot%202022-10-10%20at%2021.54.46.png)

The first guarantees that you consider every possible state. The second guarantees that the policy eventually becomes greedy.
An example algorithm is that $\epsilon$-greedy is GLIE if $\epsilon$ reduces to $0$ with a hyperbolic schedule, i.e. as $\epsilon_k=1/k$.

We can construct an algorithm from this, called GLIE Monte-Carlo Control:

![](_attachments/Screenshot%202022-10-10%20at%2021.57.13.png)

(Note that this is a stationary estimator).

What's interesting here is that our running mean estimates $Q(S_t,A_t)$ are taken over a continually changing policy, $\pi$. This is fine, since our policy is becoming more and more greedy, meaning that we accumulate increasing amount of information on the policy that we really care about. The past is eventually dominated by our new policies. 

We can prove that:

![](_attachments/Screenshot%202022-10-10%20at%2021.57.37.png)

This is our first full solution! We can apply it to *any* MDP, and it is guaranteed to find the optimal solution.

## On-Policy Temporal Difference Learning
### MC vs TD Control
Just like last lecture, we now consider TD learning. Let's remind ourselves of its benefits:

* Lower variance & better efficiency
* Can be used online
* Can be used with incomplete episodes

### Sarsa
The natural idea is to use the same generalised policy iteration strategy, but to use TD instead of MC in our control loop:

* Apply TD to $Q(S,A)$
* Use $\epsilon$-greedy policy improvement
* The only difference is that we update our value function every time step

The reason we update our value function at every time step is that we always want the most recent policy to pick our actions from.

This algorithm is called **Sarsa**.

![](_attachments/Screenshot%202022-10-12%20at%2018.20.06.png)

We begin in some state-action pair. We sample from our environment, to see a reward $R$ and new state $S'$. We then sample from our policy to get a new action, $A'$.

The update pattern we use is described at the bottom of the figure.
We now take these Sarsa updates, and insert them into our generalised policy iteration framework:

![](_attachments/Screenshot%202022-10-12%20at%2018.23.28.png)

In practice, we can think of this as meaning that at each time step, we store a table of $Q(s,a)$ values. Every step, upon taking an action we either take the optimal policy according to this table, or take a random action to explore our environment further.

What would an algorithm look like? Here is some pseudocode to help understand:

![](_attachments/Screenshot%202022-10-12%20at%2018.27.06.png)

Does this actually work? Just like GLIE Monte-Carlo, this version of Sarsa will converge to the optimal policy. The only thing we require is a GLIE policy, and we must make sure our step sizes satisfy the conditions below:

![](_attachments/Screenshot%202022-10-12%20at%2018.30.27.png)

The first says that we must make our steps sufficiently large such that we can change $Q(S,A)$ by an arbitrary amount. The second states that our changes to $Q(S,A)$ must become smaller and smaller in the limit.

In practice, we often don't have to obey these conditions and Sarsa will still converge!

Let's consider an example. We'll look at a Windy Gridworld:

![](_attachments/Screenshot%202022-10-12%20at%2018.33.13.png)

The aim is to move from S to G. We can take any of the king's moves operations. The only special thing is that every time we take a step, the environment pushes us "up" by a value indicated on the x-axis.

Using a naïve version of Sarsa does find the optimal policy:

![](_attachments/Screenshot%202022-10-12%20at%2018.35.35.png)

The first episode took 2000 episodes to complete. But the very next episode, it gets much faster. Eventually, the gradient stabilises to reflect the optimal path length. But it still won't be completely optimal due to our $\epsilon$-exploration.

Let's now consider the spectrum between Monte-Carlo and Sarsa. We call this **$n$-step sarsa**. 

![](_attachments/Screenshot%202022-10-12%20at%2018.38.30.png)

This is directly comparable to the $n$-step TD learning we had in last lecture. 

### Sarsa($\lambda$)
We now consider an algorithm that is robust to choice of $n$, and can average over many $n$: **forward-view Sarsa($\lambda$)**.

![](_attachments/Screenshot%202022-10-12%20at%2018.42.15.png)

This is another very well-known algorithm. Just like TD($\lambda$), it is equivalent to TD(0) when $\lambda=0$, and MC when $\lambda=1$. But it cannot be run online.

We can also consider the **backwards view**. The idea is again to use eligibility traces in an online algorithm. But Sarsa($\lambda$) has one eligibility trace for each *state-action pair*:

![](_attachments/Screenshot%202022-10-12%20at%2018.48.20.png)

$Q(s,a)$ is updated for **every** state $s$ and action $a$, at **every** time step. But by an amount proportional to its eligibility trace:

![](_attachments/Screenshot%202022-10-12%20at%2018.51.31.png)

Let's reiterate the intuition here: at each state transition, we observe a new reward and compute a TD-error $\delta_t$. The amount of "blame" that we assign for this $\delta_t$ for a previous state is proportional to its eligibility trace (which combines both *frequency* and *recency* heuristics).

Again, it is useful to look at pseudocode for this algorithm:

![](_attachments/Screenshot%202022-10-12%20at%2018.53.44.png)

At the beginning of each episode, we initialise our eligibility traces to $0$. Note that for the inner-most loop ("For all $s\in\mathcal{S},a\in \mathcal{A}(s)$"), we are updated the $Q$ value and eligibility trace for **all** state-action pairs. Not just the state-action pair we visit at one step of an episode.

What might this look like? Let's consider a grid-world example again.

![](_attachments/Screenshot%202022-10-14%20at%2019.51.27.png)

The aim is to try and get to some goal state. This could be like the windy grid-world we looked at above. In Sarsa, we only update the value function of our penultimate state. We only propagate the value of our state back by one step.

In Sarsa($\lambda$), the $\lambda$ parameter determines how far back the value propagates back along the trajectory taken. So when we see the final reward, all states we took are updated. You get a much faster propagation of information backwards through time. $\lambda$ overcomes the **"tyranny of the time-step"**.

One subtlety here: we are running updates of our $q$ values here at *every single step*. But we don't see any reward until the last state. Only at the final point does information propagate backwards.

## Off-Policy Learning
There are many cases where we wish to evaluate a target policy $\pi(a|s)$ (to compute $v_\pi(s)$ or $q_\pi(s)$) while following behaviour policy $\mu(a|s)$.
Some reasons this might be important are:

* Learn from observing humans or other agents
* Re-use experience generated from old policies $\pi_1, \pi_2,...,\pi_{t-1}$. This allows us to re-use data from old episodes for which we were using a different policy.
* Learn about *optimal* policy while following an *exploratory* policy. I.e. we can generate a policy that does as much exploration as we want, whilst still learning about the optimal policy (which will ultimately be deterministic).
* Learn about *multiple* policies while following *one* policy; sometimes we care about the outcomes of several different trajectories through an environment.

### Importance Sampling
We will look at two methods for doing off-policy learning. The first is **importance sampling**. 
The main idea here is to estimate the expectation of a different distribution:

![](_attachments/Screenshot%202022-10-14%20at%2020.08.02.png)

We can apply importance sampling to MC by doing it along the entire trajectory. At every step, there is an action we took according to our behaviour policy. And there's a probability that I would have taken the same action under the policy that I care about:

![](_attachments/Screenshot%202022-10-14%20at%2020.10.52.png)

I think this is intuitively quite simple: we weight the return $G_t$ by a ratio that reflects how likely it would have been to have seen $G_t$ under our target policy.

In general, this weighting factor becomes vanishingly small. In addition, it is **extremely high variance**; **MC does not work off-policy**.

Instead, it's best to importance sample with TD learning:

![](_attachments/Screenshot%202022-10-14%20at%2020.13.31.png)

This means we only need a single importance sampling correction (because we bootstrap over one step). In other words, over the one step that we take, we correct for whether our target policy matches the step we actually took.
This is much lower variance than Monte-Carlo.

### Q-Learning
However, the idea that works best with off-policy learning is Q-learning. 
It's an alternative to importance sampling. **It is specific to TD(0)**. 

We consider a specific case where we consider our $Q$ values. 

The idea is to choose our next action using the behaviour policy, i.e. $A_{t+1}\sim \mu(\cdot|S_t)$. We *also* consider an alternative successor action that we might have taken if we followed our target policy, $A'\sim \pi(\cdot|S_t)$.
We then update $Q(S_t,A_t)$ towards the value of the alternative action:

![](_attachments/Screenshot%202022-10-14%20at%2020.19.12.png)

To be clear: we were in state $S_t$. We took action $A_t$. We update $Q(S_t, A_t)$ in the direction of  the reward $R_{t+1}$ plus the value of the *alternate* action that we actually care about. 

#### Q-Learning Control Algorithm
There is a special case of this, which is the well-known Q-learning algorithm. If you hear about Q-learning, it refers to the following.

It's the special case where the target policy is a greedy policy. We allow both the behaviour and target policies to improve. The target policy is greedy w.r.t. $Q(s,a)$:

![](_attachments/Screenshot%202022-10-14%20at%2020.25.14.png)

But the behaviour policy $\mu$ is $\epsilon$-greedy w.r.t $Q(s,a)$. Plugging this in to the above equation then simplifies:

![](_attachments/Screenshot%202022-10-14%20at%2020.26.25.png)

We can visualise this with a diagram:

![](_attachments/Screenshot%202022-10-14%20at%2020.26.58.png)

And we can prove that:

![](_attachments/Screenshot%202022-10-14%20at%2020.27.14.png)

The pseudocode is:

![](_attachments/Screenshot%202022-10-14%20at%2020.27.47.png)

## Summary 

### Relationship between DP and TD

![](_attachments/Screenshot%202022-10-14%20at%2020.29.31.png)

The RHS is a sample of the central column. The LHS is the type of Bellman equation that is used.

