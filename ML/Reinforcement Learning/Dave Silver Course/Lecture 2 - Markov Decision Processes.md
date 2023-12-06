I didn't go through the extension content of this lecture.

```toc
```

## Markov Processes
### Intro
The MDP provides a formal description of an environment, that can be used for RL. We will begin with the case where the environment is fully observable. We can think of this as meaning that the state completely characterises the process.

Almost all RL problems can be formalised as MDPs. Including continuous processes. Partially observable problems can be converted into MDPs.

### Markov Property
The central idea to the MDP is the **Markov Property**, which we saw in the [last lecture](Lecture%201%20-%20Introduction%20to%20RL.md). As a reminder, the idea is that the *future is independent of the past, given the present*. A state is *Markov* iff:

$P[S_{t+1}|S_t]=P[S_{t+1}|S_1,\dots,S_t]$

This means that the state captures all relevant information from the history. Once the state is known, the history can be discarded.

For a Markov state $s$ and successor $s'$, the **state transition probability** is defined by: 

$P_{ss'}=P[S_{t+1}=s'|S_t=s]$

The **state transition matrix** $\mathcal{P}$ defines transition probabilities from all states $s$ to all successor states $s'$,

![](_attachments/Screenshot%202022-09-05%20at%2017.25.32.png)

where each row of the matrix sums to 1. This matrix tells you the complete structure of a Markov problem.

### Markov Chains
A **Markov Process** is a memoryless random process, i.e. a sequence of random states $S_1, S_2,\dots$ with the Markov property. We can think of it as a tuple $<\mathcal{S},\mathcal{P}>$, where:
*  $\mathcal{S}$ is a (finite) set of states
* $\mathcal{P}$ is a state transition probability matrix (as above)

Let's consider a concrete example:

![](_attachments/Screenshot%202022-09-05%20at%2017.32.30.png)

Here, the sleep state represents a **terminal** process. 
What does it mean to take a *sample* from this process? A sample is just a set of possible events, sampled from the given dynamics:

![](_attachments/Screenshot%202022-09-05%20at%2017.33.43.png)

Note that the sum of probabilities for paths emerging from each node is 1 (this is equivalent to each row of the transition matrix summing to 1). The transition matrix for this problem would be:

![](_attachments/Screenshot%202022-09-05%20at%2017.34.07.png)


## Markov Reward Processes
So far we haven't talked about RL at all. Let's now add some of this machinery.

### MRP
The first step is to add rewards. A **Markov Reward Process** is a Markov chain with values:

![](_attachments/Screenshot%202022-09-05%20at%2017.38.07.png)

Going back to our student example:

![](_attachments/Screenshot%202022-09-05%20at%2017.40.25.png)

### Return 
What we care about is the *total* reward that we will get across some chain. To do so, we use a quantity called the **return**. The return $G_t$  for a single (random) sample is the total discounted reward from time-step $t$:

![](_attachments/Screenshot%202022-09-05%20at%2017.42.03.png)

The **discount** $\gamma \in [0,1]$ is the present value of future rewards. This values immediate rewards above delayed reward. This is a way of controlling the return (i.e. ensuring it is finite).
$\gamma$ close to $0$ leads to **myopic** evaluation.
$\gamma$ close to 1 leads to **far-sighted** evaluation.

But why discount?
* It's mathematically convenient
* Uncertainty about the future may not be fully represented. I.e. we may not trust our model fully. If we think we will get a 10x return in 10 years, but we don't fully trust our model, then it makes sense to discount.

It is sometimes possible to use *undiscounted* MDPs (i.e. $\gamma=1$), e.g. if all sequences terminate.

### Value Function
The **value function** gives the long-term value of state $s$. Formally, it is just the expected return starting from state $s$:

![](_attachments/Screenshot%202022-09-05%20at%2017.51.31.png)

We have an expectation because the environment is stochastic: we can't predict the set of states we will move through in the future.

Let's go back to our previous example:

![](_attachments/Screenshot%202022-09-05%20at%2017.53.01.png)
(Note that the subscripts above are timesteps).

We will see how to compute the value for a state later in the lecture.

### Bellman Equation
The Bellman Equation is one of the most fundamental relationships in RL. It is relevant in several other areas, e.g. dynamic programming.

The value function can be decomposed into two parts:
1. Immediate reward $R_{t+1}$
2. Discounted value of successor state $\gamma v(S_{t+1})$

In full:

![](_attachments/Screenshot%202022-09-05%20at%2017.58.55.png)

The Bellman Equation is a bit of a tautology; if we have a correct value function, it simply *must* satisfy this property. One way to understand this is with a **backup diagram**:

![](_attachments/Screenshot%202022-09-06%20at%2016.27.50.png)

We can think of it as a one-step look-ahead search. We begin in state $s$. We can then do a weighted sum over the value functions at possible future states, and that determines the value function of our current state. This can be written as:

![](_attachments/Screenshot%202022-09-06%20at%2016.29.51.png)

Let's look back at our concrete example. We were previously just given the value function of the MDP, but we can now verify that it is indeed a value function since it satisfies the Bellman equation:

![](_attachments/Screenshot%202022-09-06%20at%2016.31.15.png)

There is a slightly subtlety here, in that we assume we get a reward as we *exit* a state. This is just a convention, but explains the indexing used in most of these lecture slides.

The Bellman equation can be expressed more concisely with matrix notation:

![](_attachments/Screenshot%202022-09-06%20at%2016.39.35.png)
(bottom-left should be $\mathcal{P}_{n1}$).

The value and reward functions are written as vectors.

The Bellman equation is a linear equation, so we can **solve directly for our value function**:

![](_attachments/Screenshot%202022-09-06%20at%2016.41.49.png)

This assumes our transition matrix is small enough to invert. The computational complexity in doing so is $O(n^3)$, which isn't usually feasible.
There are many iterative methods for large MRPs:

* Dynamic Programming
* Monte-Carlo evaluation
* Temporal Difference Learning

These are all covered in future lectures.

## Markov Decision Processes
The prior sections have all been the building blocks to talking about MDPs. These are what we actually use in RL.
### MDP
An MDP is a Markov Reward process, with actions. It is an *environment* in which all states are Markov.

![](_attachments/Screenshot%202022-09-06%20at%2016.44.59.png)

The key difference to MRPs is that the transition probability matrix is now dependent on our actions.

Let's reconstruct our student MRP as an MDP:

![](_attachments/Screenshot%202022-09-06%20at%2016.50.39.png)

The black node in this graph represents that *pub* is now an action (instead of a state). The main idea here is that the state transition matrix is now dependent on our action. If we choose to go to the pub, there is now **randomness**. The goal is to now choose actions such that our reward is maximised.

We can imagine that all other black arcs possess an action node, but that they're fully deterministic. In practice, an action often maps stochastically to states (e.g. wind when flying a helicopter).

The key difference between an MDP and an MRP is that we now have some degree of control over transitioning between states; since we can take actions, we can begin to look for a *policy* (albeit a stochastic one) that maximises future reward.

### Policies
To do so, we need to formalise how we make decisions.
A **policy** $\pi$ is a distribution over actions, given states:

![](_attachments/Screenshot%202022-09-06%20at%2016.53.15.png)

This is a **stochastic policy**. It's just a stochastic transition matrix. The stochasticity is useful since it gives us the ability to do exploration.

A policy completely defines the behaviour of an agent. In an MDP, policies depend on the current state (not the history), meaning they're **stationary** (time-independent).

A useful point to note is that, given an MDP and a fixed policy, we can always recover an MRP. The state and reward sequence $S_1, R_2, S_2,\dots$  is an MRP $<\mathcal{S}, \mathcal{P}^{\pi}, \mathcal{R}^{\pi}, \gamma>$ where:

![](_attachments/Screenshot%202022-09-06%20at%2017.00.11.png)

### Value Functions
Our value function is clearly a function of our policy. This is why the value function $v_{\pi}$ is subscripted by $\pi$. It essentially says how good state $s$ is, given a policy $\pi$.

![](_attachments/Screenshot%202022-09-06%20at%2017.03.32.png)

We can also use this to determine which action is best to take:

![](_attachments/Screenshot%202022-09-06%20at%2017.03.54.png)

To be clear: we follow policy $\pi$ *after* we have taken action $a$. So our policy isn't used to make decisions in that first step.

The state-value function for our concrete example, given a policy where we chose things with 50-50 chance, is:

![](_attachments/Screenshot%202022-09-06%20at%2017.06.17.png)

### Bellman Expectation Equation
For MDPs, we can formulate a Bellman Equation using the same idea as before: we decompose our current value function as the sum of immediate reward and the value function of the next state:

![](_attachments/Screenshot%202022-09-06%20at%2017.07.53.png)

The action-value function can be similarly decomposed:

![](_attachments/Screenshot%202022-09-06%20at%2017.08.11.png)

Let's look at this pictorially. White circles represent states; black circles represent actions. If we are in some state $s$, then we can take some action $a$, which has an associated action value function $q_{\pi}$. 

![](_attachments/Screenshot%202022-09-06%20at%2017.12.20.png)

This can be written as:

![](_attachments/Screenshot%202022-09-06%20at%2017.14.21.png)

What happens if we start at an action instead? We now have to average over the dynamics of our MDP:

![](_attachments/Screenshot%202022-09-06%20at%2017.14.52.png)

We can again average:

![](_attachments/Screenshot%202022-09-06%20at%2017.15.25.png)

In short: $v$ tells us how good it is to be in a particular state; $q$ tells us how good it is to take a particular action from some state.

We can now put these two togther:

![](_attachments/Screenshot%202022-09-06%20at%2017.16.28.png)

This gives us a recursion relation that means we can understand $v_{\pi}$ :

![](_attachments/Screenshot%202022-09-06%20at%2017.17.01.png)

We are averaging in two ways now: over our policy, and over our transition probabilities (i.e. the environment dynamics).

We can do exactly the same thing for action values.

![](_attachments/Screenshot%202022-09-06%20at%2017.18.54.png)

Key point: in all these cases, the fundamental idea is very simple. **The value function at the current time step is given by the immediate reward, plus the value function of the state in which you end up**.

Let's use the Bellman equation for the state-value function to verify our concrete example is indeed a value function:

![](_attachments/Screenshot%202022-09-06%20at%2017.20.29.png)

### Optimal Value Functions
Let's now talk about the actual problem we care about: **finding the best behaviour in the MDP.**

![](_attachments/Screenshot%202022-09-06%20at%2017.28.17.png)

We can do the same for action-value functions:

![](_attachments/Screenshot%202022-09-06%20at%2017.28.42.png)

I.e. there are various ways that we can traverse the system; we care about choosing a policy that maximises the expected reward. 
The optimal value function specifies the best possible performance in the MDP. **An MDP is solved when we know the optimal value function**.

Looking at our concrete example, the optimal value function looks like:

![](_attachments/Screenshot%202022-09-06%20at%2017.33.38.png)

And the optimal action-value function looks like:

![](_attachments/Screenshot%202022-09-06%20at%2017.34.02.png)

We can also consider the **optimal policy**. **This is what we really care about**. We first define a partial ordering over policies:

![](_attachments/Screenshot%202022-09-06%20at%2017.36.16.png)

Note that this must apply across **all states**. There exists an important theorem for MDPs that states that **for any MDP, there exists a policy that is better than (or equal to) all others**:

![](_attachments/Screenshot%202022-09-06%20at%2017.38.10.png)

These last two statements are a tautology; since the $v_*$ is defined as that being achieved by the optimal policy, the optimal policy clearly achieves the optimal value function!

To find an optimal policy, you solve for $q_*$ and pick the action that maximises $q_*$:

![](_attachments/Screenshot%202022-09-07%20at%2008.58.06.png)

So if we know $q_*(s,a)$ then we immediately have the optimal policy. There is always a deterministic optimal policy for any MDP. 

Let's look at our concrete example again:

![](_attachments/Screenshot%202022-09-07%20at%2008.59.41.png)

Remember that the black arcs represent actions we can take. The red arcs show the optimal policy. We can see that this must be the case, given the $q_*$ values.

### Bellman Optimality Equation
The **Bellman Optimality Equation** is the one that really tells you how to solve your MDP. If you open up a textbook and it mentions the "Bellman Equation", it will mean this one. It gives a relationship for $v_*$.

The optimal value functions are recursively related:

![](_attachments/Screenshot%202022-09-07%20at%2009.02.56.png)

Instead of picking the mean of different possible actions, we take the one that maximises our $q_*$. So the value of a state is the max of the $q_*$ that you can get from that state:

![](_attachments/Screenshot%202022-09-07%20at%2009.04.17.png)

Doing the same for the other half, i.e. $q_*$:

![](_attachments/Screenshot%202022-09-07%20at%2009.05.02.png)

We are now looking at what the *environment* might do. I.e. once we take an action, what state might we end up in? 

The optimal value function is simply the immediate reward, plus the value function of some future state:

![](_attachments/Screenshot%202022-09-07%20at%2009.05.38.png)

Putting these together:

![](_attachments/Screenshot%202022-09-07%20at%2009.06.35.png)

(Where the max operator applies to the whole expression). We can do the same for $q_*$:

![](_attachments/Screenshot%202022-09-07%20at%2009.07.11.png)

For our concrete example:

![](_attachments/Screenshot%202022-09-07%20at%2009.08.17.png)

The Bellman Optimality Equation is non-linear (it contains a $\max$). In general, there is no closed form solution. So we use **iterative solution methods**, of which the best-known examples are:

* Value Iteration (dynamic programming)
* Policy Iteration (dynamic programming)
* Q-learning
* Sarsa

## Extensions to MDPs
I didn't make notes on this section. It wasn't really covered in the lecture. Topics include: 

* Infinite MDPs
* POMDPs
* Average Reward MDPs


