I didn't make notes on contraction mapping, or approximate dynamic programming, as these weren't covered in the lecture
```toc
```

## Introduction
In this lecture, we do not try to solve the full RL problem - we assume the transition probabilities and rewards are given. We instead look at the **planning** problem.

<u>What is Dynamic Programming?</u>
One of the oldest solutions to optimal control is **dynamic programming**.

What is dynamic programming? **Dynamic** implies some kind of sequential/temporal component to the problem.
By **programming**, we mean *mathematical* programming (**not** the same as writing a C program, etc). This means to optimise a **program**, i.e. a policy.

Dynamic programming lets us solve a problem by breaking it into subproblems. We then:

1. Solve the subproblems;
2. Combine their solutions.

<u>Requirements for dynamic programming</u>
This is a very general solution method for problems which satisfy two properties:

1. **Optimal substrcture**
	* *Principle of optimality applies*
	* Means that the optimal solution can be decomposed into subproblems; the solution to the subproblems tells us the solution to the overall problem. The canonical example of this is finding the shortest distance between two points.
2. **Overlapping subproblems**
	* The subproblems that occur do so many times
	* Solutions can be cached and resused. This means we actually gain something by breaking the problem into subproblems - it is more efficient.

MDPs satisfy both of these properties. The Bellman Equation gives recursive decomposition. The value function stores and reuses solutions - it's like a cache of all the good information we have found about the MDP.

<u>Planning by dynamic programming</u>
Dynamic programming assumes full knowledge of the MDP. It is used for **planning** in an MDP. There are two components to planning: **prediction**, and **control**.

For prediction:
* We're given an MDP and policy $\pi$
* It outputs a value function $v_\pi$

For control:
* Input is an MDP
* The output is the optimal value function $v_*$ and an optimal policy $\pi_*$

<u>Other Applications of Dynamic Programming</u>

* Scheduling algorithms
* Graph algorithms (e.g. shortest path algorithms)
* Bioinformatics (e.g. lattice models)

## Policy Evaluation
### Iterative Policy Evaluation
The first key component is how to evaluate a given policy.

One solution to this is an iterative application of Bellman expectation equation. 

To solve this, we begin with some arbitrary intial value vector, $v_1$. We might initialise this to $0$, i.e. we assume the value of *every state* is $0$. We then iteratively improve our estimates: $v_1\to v_2\to \dots\to v_\pi$

We do this using **synchronous backups**. At each iteration $k+1$, and for *all* states $s \in \mathcal{S}$, we update $v_{k+1}(s)$ using $v_k(s')$, where $s'$ are the successor states of $s$.

We will discuss **asynchronous** backups later. It is also possible to prove that this converges to a value function, $v_\pi$. This is proved at the end of the lecture.

So how does this actually work?
Consider the Bellman Expectation Equation:

![](_attachments/Screenshot%202022-09-11%20at%2011.55.37.png)

A reminder: the value of the root is given by a one-step look-ahead. A true value function must satisfy this. 

**We can turn this equation into an iterative update. Intiutively, to compute the value function at iteration $k+1$ for state $s$, we use the value functions from iteration $k$ for successor states $s'$.**
We do this for every single state in the MDP.

I initially thought that we had to begin at the leaf nodes of the MDP and work our way towards the root during each iteration. This is not correct. We can compute the new value function for all states simultaneously (synchronous backups). The following example helps illustrate this.

### Example: Small Gridworld
Let's consider a simple example:

![](_attachments/Screenshot%202022-09-11%20at%2012.00.41.png)

<u>Rules:</u>
The grey shaded squares are terminal states.
If you take an action that would take you off the grid, it simply has no effect and you remain in the same square.
We get $-1$ reward on all transitions. The aim is to minimise how long it takes us to get to a grey square.

The agent follows a uniform random policy:

![](_attachments/Screenshot%202022-09-11%20at%2012.02.39.png)

The question we wish to answer for the *planning* problem is: given this policy, what's our expected cumulative reward? Consider the following diagram (ignore the RH column for now):

![](_attachments/Screenshot%202022-09-11%20at%2012.03.30.png)

The squares containing $-1.7$ should actually show $-1.75$.

The left column shows a state-value function. We begin by initialising with $0.0$ everywhere.
The output of $k=1$ uses the initial values. We can see that the only states that have $0.0$ reward are the terminal states. 

To be clear: the values at each iteration are computed using the Bellman Expectation Equation. (A useful exercise here is to justify why each value takes the value that it does).

And so on:

![](_attachments/Screenshot%202022-09-11%20at%2012.07.53.png)

The right-hand column shows us that we can construct a better policy (instead of a random walk), by using the values functions for each state. We can see that the optimal policy is obtained far before the value function reaches convergence.

## Policy Iteration
<u>How to Improve a Policy</u>
Given a policy $\pi$, how can we improve on it? If we had a process to do so, we could apply it iteratively and eventually land on the optimal policy.
There are two stages:

1. Evaluate a policy $\pi$
2. Improve the policy by acting greedily wrt $v_\pi$

In the small gridworld example, the improved policy was optimal, i.e. $\pi'=\pi^*$. But in general, we may need multiple iterations of improvement/evaluation.

This process of **policy iteration always converges to $\pi^*$ and $v^*$**

The following illustration helps to explain this process:

![](_attachments/Screenshot%202022-09-11%20at%2017.56.17.png)

The up arrows represent policy evaluation. The down arrows represent (greedily) obtaining a new policy. 

### Example: Jack's Car Rental
We'll now consider a toy example of **car rental**. 
In this setup, we have two locations. Cars are rented from each of these two locations.
There's a random request/return process. But the rates of these are different in the two locations.
We can frame this as an RL problem:

States: there's a maximum of 20 cars in each location.
Actions: we can move up to 5 cars between locations overnight.
Reward: $10 for each car rented (but it must be available at the correct location)
Transitions:
* Poisson distribution, with $n$ returns/requests.
* 1st location: average requests = 3, averages returns = 3
* 2nd location: average requests = 4, average returns = 2

What does policy iteration look like in this case?

![](_attachments/Screenshot%202022-09-11%20at%2018.03.15.png)

These policies indicate how many cars to move given a state. 
We begin with $\pi_0$. We can then construct a value surface (through iterative policy evaluation) that gives expected reward for any state. We can then define a new policy that acts greedily wrt this surface.  $\pi_4$ is the optimal solution.

After 1 iteration, it's clear that we should generally be shifting cars to the location with fewer cars.

### Policy Improvement
Let's consider this process a bit more formally.
Assume a deterministic policy, $\pi(s)$
We can improve the policy by acting **greedily**.
Considering just **one time step**:

![](_attachments/Screenshot%202022-09-11%20at%2018.11.33.png)

We can show that this improves the value from any state $s$ (to reiterate, this is over one step):

![](_attachments/Screenshot%202022-09-11%20at%2018.12.46.png)

(assuming we follow policy $\pi$ after the first step).

It therefore improves the value function, i.e  $v_{\pi'}(s) > v_\pi(s)$:

![](_attachments/Screenshot%202022-09-11%20at%2018.14.49.png)

So this shows that acting greedily is at as useful as the previous policy we had. What happens if this process stops? In other words, we have equality:

![](_attachments/Screenshot%202022-09-11%20at%2018.16.44.png)

Then the *Bellman optimality equation* has been satisfied:

![](_attachments/Screenshot%202022-09-11%20at%2018.17.01.png)

Therefore $v_\pi(s)=v_*(s)$ for all $s\in\mathcal{S}$. So $\pi$ is an optimal policy.

It is quite interesting that this algorithm is guaranteed not to reach a local maximum. This can be explained by a **contraction mapping** argument.

### Extensions to Policy Iteration
<u> Modified Policy Iteration</u>
Does policy evaluation need to converge to $v_\pi$? We saw in the small gridworld example that the optimal policy was achieved before this stage.

We could instead introduce a stopping condition, for instance some $\epsilon$-convergence of value function. Or simply stop after $k$ iterations of iterative policy evaluation?

We could even update policy every iteration, i.e. stop after $k=1$. This is in fact equivalent to [value iteration](#^946404).

## Value Iteration

^946404

### Value Iteration in MDPs
<u>Principle of Optimality</u>
Let's quickly return to the principle of optimality. Any optimal policy can be subdivided into two components:

* An optimal first action $A_*$
* An optimal policy from successor state $S'$

The **Principle of Optimality** states that:

![](_attachments/Screenshot%202022-09-11%20at%2018.28.15.png)

In other words, a policy is optimal iff it reaches the optimal value for any state $s'$.

<u>Deterministic Value Iteration</u>
If we know the solution to subproblems $v_*(s')$, then the solution $v_*(s)$ can be found by one-step lookahead

![](_attachments/Screenshot%202022-09-11%20at%2018.33.42.png)

The idea of value iteration is to apply these updates iteratively. The intuition is that we start with the final rewards, and work backwards. This still works with stochastic MDPs, and those that contain loops.

Let's consider the example of a **shortest path** algorithm, in another small gridworld:

![](_attachments/Screenshot%202022-09-11%20at%2018.35.36.png)

Intuitively, once we know the final reward is $0$, we can work our way back and iteratively determine the value of squares further and further away from the optimal state.

We can summarise this algorithm more formally as follows:

![](_attachments/Screenshot%202022-09-11%20at%2018.41.42.png)

To reiterate: we do not consider any policies here. We instead work solely in value space. As such, the values we obtain might not even be the values associated with any real policy. But at the end of this process, we know that we have the value function for the optimal policy.

For completeness, let's formulate value iteration with a backup diagram. For each iteration, every state has a turn in being the root of this diagram:

![](_attachments/Screenshot%202022-09-11%20at%2018.49.02.png)

### Summary of DP Algorithms
Here's a summary of the algorithms we've covered so far:

![](_attachments/Screenshot%202022-09-11%20at%2018.51.52.png)

In all cases, we are trying to solve the planning problem. There are two types of planning problem: prediction and control.

What we have also seen is that the third line is a special case of the second; when using policy iteration with $k=1$, we recover value iteration. To be super clear: value iteration is a special case of policy iteration, where do just one step of policy evaluation before updating our policy.

## Extensions to Dynamic Programming
### Asynchronous Dynamic Programming
The methods described so far used synchronous backups. I.e. all states are backed up (used as the root of the backup diagram) in parallel. Asynchronous DP backes up states individually, in any order.

This can significantly reduce computation. We can still guarantee convergence if all states continue to be selected.

There are three simple ways we can apply asynchronous dynamic programming:

1. **In-place** dynamic programming
2. **Prioritised sweeping**
3. **Real-time** dynamic programming

**In-place DP** is motivated by the observation that synchronous value iteration stores two copies of the value function.

![](_attachments/Screenshot%202022-09-11%20at%2019.03.23.png)

For in-place DP, we immediately overwrite our old value function, in whatever order we visit states:

![](_attachments/Screenshot%202022-09-11%20at%2019.04.20.png)

This means we will be using updated version of the value function within the same iteration.
This tends to be more efficient. The only remaining thing to figure out is the order in which to visit states.

This motivates **prioritised sweeping**. The idea is to measure how important it is to update a state in your MDP. Prioritised sweeping uses a priority queue to establish which states to update.

Most use the magnitude of the Bellman error to guide state selection:

![](_attachments/Screenshot%202022-09-11%20at%2019.07.29.png)

In other words, we order the priority by *how much* the state's value is going to change.

The third idea is **real-time DP**. The intuition is to only update states that the agent actually visit. We run an agent, and use its experience to guide the selection of states. 

### Full-width and Sample Backups
Dynamic programming uses *full-width* backups. This means that for each backup, every successor state and action is considered. In other words, we are considering the entire branching factor. 

For large problems this suffers from the **curse of dimensionality**. The number of states $n$ grows exponentially with the number of state variables.

In subsequent lectures, we will consider **sample backups**. We start in some state, sample an action according to our policy, sample a transition according to our dynamics, and then do backup. This means we don't need a model of our environment:

![](_attachments/Screenshot%202022-09-11%20at%2019.15.27.png)

### Approximate Dynamic Programming

## Contraction Mapping