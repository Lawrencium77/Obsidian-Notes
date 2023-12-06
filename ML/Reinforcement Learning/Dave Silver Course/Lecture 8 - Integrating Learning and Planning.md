```toc
```

# Introduction
This lectures focuses on fundamental ideas & architectures in RL, and how to combine them. 

[Last lecture](Lecture%207%20-%20Policy%20Gradient%20Methods.md) considered how to learn a **policy** directly from experience. Previous lectures focused on learning the **value function** directly from experience.
This lecture considers how to learn a **model** directly from experience. We can then use this model to **plan** and thus construct a value function and/or policy.

As a reminder, model-free RL looks something like:

![](_attachments/Screenshot%202022-11-05%20at%2018.41.42.png)

Whereas in model-based RL, we replace the "real" world with a simulated version of the environment:

![](_attachments/Screenshot%202022-11-05%20at%2018.41.52.png)

# Model-Based RL
Up until now, this course has focused on model-**free** RL. We now consider **model-based** RL.
The underlying process can be illustrated like so:

![](_attachments/Screenshot%202022-11-05%20at%2018.43.41.png)

We begin with some experience; we use it to build a model; we use the model to plan; and we then use the value function/policy to make actions.
And so on.

Another way of interpreting this: model learning learns an MDP, and planning solves that MDP. 

Let's consider the pros/cons of model-based RL:

![](_attachments/Screenshot%202022-11-05%20at%2018.46.09.png)

Sometimes, learning a model can simpler than learning a value function. Consider chess: there are $\sim 10^{40}$ board positions, so learning a policy is very hard. But the model is extremely simple. So if we can learn the model, then we can exploit tree search.

There is also a natural supervision signal that makes it easier to learn the model.

## Learning a Model
What is a model? This is described below:

![](_attachments/Screenshot%202022-11-05%20at%2018.50.37.png)

Both of these are one-step models. 
We typically assume that there is conditional independence between state transitions and rewards. This means that we can learn $\mathcal{P}_\eta$ and $\mathcal{R}_\eta$ independently:

![](_attachments/Screenshot%202022-11-05%20at%2018.51.32.png)

The following figure describes what it means to **learn** a model:

![](_attachments/Screenshot%202022-11-05%20at%2018.54.57.png)

To be clear: learning the reward is a **regression** problem - how much reward do we expect to get? Learning the transition probabilities is a **density estimation** problem -  we learn probability densities over states.

We then pick an appropriate loss function (e.g. MSE, KL divergence), and find the parameters $\eta$ that minimise the empirical loss. 
Let's consider the simplest type of model: a **table-lookup** model. The model is an explicit representation of the MDP, $\hat{\mathcal{P}}, \hat{\mathcal{R}}$. We can model transitions and rewards by taking the average:

![](_attachments/Screenshot%202022-11-05%20at%2019.02.00.png)

Alternatively, we could just record our experience, and sample from it:

![](_attachments/Screenshot%202022-11-05%20at%2019.02.59.png)

These two approaches give you the same information. The first is parametric, and the second is non-parametric.

Let's make this concrete:

![](_attachments/Screenshot%202022-11-05%20at%2019.04.09.png)

To be clear: we can take the 8 episodes of experience, and use it to construct a table lookup model.

## Planning with a Model
How do we actually plan? Well, we need to solve the MDP represented by our model. We can do this using any of our favourite planning algorithms: Value Iteration, Policy Iteration, Tree Search, etc.

A method we will talk a lot in this lecture is **sample-based planning**. The idea is to use the model **only** to generate samples. We then sample experience from the model:

![](_attachments/Screenshot%202022-11-05%20at%2019.07.15.png)

We can then apply our familiar **model-free** RL strategies to the samples. E.g. Monte-Carlo control, Sarsa, Q-learning.
Sample-based planning methods are often most efficient (when compared to dynamic programming methods applied to a model).

Let's go back to our AB example:

![](_attachments/Screenshot%202022-11-05%20at%2019.08.27.png)

It's clear that the advantage of this approach is that we can sample *way* more trajectories than we had in our initial "real" experience. We essentially have infinite data.

Clearly, having an imperfect model can effect the performance of this algorithm. This is summarised below:

![](_attachments/Screenshot%202022-11-05%20at%2019.12.04.png)


# Integrated 
This section considers combining both model-free and model-based methods.

## Dyna
We consider two sources of experience:

![](_attachments/Screenshot%202022-11-06%20at%2010.36.07.png)

A classic architecture that combines model-free and model-based RL is **Dyna**. The key idea is that we use both our real and simulated experience to learn our value function or policy. These ideas are summarised as follows:

![](_attachments/Screenshot%202022-11-06%20at%2010.40.29.png)

Here's a new version of the model-based RL loop we had above:

![](_attachments/Screenshot%202022-11-06%20at%2010.40.57.png)

The canonical Dyna algorithm is **Dyna-Q**:

![](_attachments/Screenshot%202022-11-06%20at%2010.41.54.png)

After every observation, we update our value function according to Q-learning **and** we update our model. We also have an inner loop where we simulate more experience. We can imagine this as a kind of "thinking" loop.

This seems a bit weird, but can make our algorithm more data efficient. Here's an example of a simple maze:

![](_attachments/Screenshot%202022-11-06%20at%2010.46.46.png)

The "planning steps" variable is our $n$, i..e how many "thinking" steps we do per iteration. The $x$-axis is the number of episodes. It's clear that Dyna-Q is much more data efficient.

# Simulation-Based Search
This last section focuses on the **planning** part of model-based RL. We explore the idea of generating simulated experience.
The key ideas we are going to use are **sampling** and **forward search**.

**Forward search** algorithms don't explore the entire state space. They put a particular focus on the **current state**, by doing a lookahead:

![](_attachments/Screenshot%202022-11-06%20at%2010.54.13.png)

The key idea is that we **don't need to solve the whole MDP. We just focus on the MDP starting from the current state**.

**Simulation-based search** is a forward-search paradigm that uses sample-based planning. We start from our current state, and "imagine" what might happen next, by sampling from our model. 
We then apply model-free RL to the simulated episodes:

![](_attachments/Screenshot%202022-11-06%20at%2010.57.23.png)

To illustrate this idea a bit further:

![](_attachments/Screenshot%202022-11-06%20at%2010.58.52.png)

In the first line, the superscript $k$ indicates which episode of experience we have simulated. 

## Monte-Carlo Search
Let's consider **Simple Monte-Carlo Search** in more detail. The algorithm looks something like the following:

![](_attachments/Screenshot%202022-11-06%20at%2011.01.15.png)

For our current state, we consider every possible future action. For each of these actions, we run a load of simulations. The average return from these simulations is used to estimate $Q(S_t,a)$.

A more advanced example is **Monte-Carlo Tree Search (MCTS)** :

![](_attachments/Screenshot%202022-11-06%20at%2011.03.23.png)

Again, we start from the root state and simulate episodes. But we now consider $\pi$ to be varying with time. 
Instead of just evaluating the root actions, we evaluate *every* state-action pair that we visit. So we build a search tree. Since this is Monte-Carlo, we take the mean of returns from each state-action pair to estimate $Q_\pi(s,a)$. 

There is one more part to this process. In MCTS, the simulation policy $\pi$ **improves**. 
After every simulation, we improve $\pi$ according to the Q-values in the search tree. 
We break up our simulation into two phases: whether we're in our tree, or if we've gone beyond our tree (so we don't have any info):

![](_attachments/Screenshot%202022-11-06%20at%2011.09.38.png)

To be clear: when we run beyond the tree, we behave according to some default, random simulation policy. 
The algorithm is that for every simulation, we evaluate our state with MC evaluation and improve our policy with $\epsilon$-greedy:

![](_attachments/Screenshot%202022-11-06%20at%2011.10.41.png)

This is exactly the same as **Monte-Carlo control**, but applied to simulated experience starting from the root state. This is guaranteed to converge on the optimal search tree, $Q(s,a)\to q_*(s,a)$.

## MCTS in Go
Here is a slide that quickly summarises the rules of Go:

![](_attachments/Screenshot%202022-11-06%20at%2011.14.42.png)

To do position evaluation, we have a reward function:

![](_attachments/Screenshot%202022-11-06%20at%2011.15.42.png)

We consider policies for both sides (so are doing **self-play**):

![](_attachments/Screenshot%202022-11-06%20at%2011.16.19.png)

**Simple** MC evaluation looks something like:

![](_attachments/Screenshot%202022-11-06%20at%2011.16.59.png)

This is a really simple way to estimate a value function. But it's highly effective. 
To turn this into a tree search algorithm, we begin by just following our default policy:

![](_attachments/Screenshot%202022-11-06%20at%2011.18.26.png)

We can use this to add information to our search tree. Let's add another iteration:

![](_attachments/Screenshot%202022-11-06%20at%2011.19.14.png)

The root node has had one win in two. For the next iteration, we use the tree policy to guide our search somewhat:

![](_attachments/Screenshot%202022-11-06%20at%2011.20.02.png)

And so forth. The key idea is that we expand the parts of the search tree that are most promising. It completely ignores the parts of the search tree that give bad results. It is important to add a degree of exploration.

Why is this a good idea?

![](_attachments/Screenshot%202022-11-06%20at%2011.22.24.png)

## Temporal-Difference Search
Monte-Carlo search is just one example of a family of search problems. But the key ideas are doing forwards search, and sampling. We can consider other model-free RL approaches.

Let's try TD learning instead!

TD search applies Sarsa to the sub-MDP from the current state.
Why should we bother?

![](_attachments/Screenshot%202022-11-06%20at%2011.27.05.png)

What does it look like? Again, we begin from our start state. For each step of simulation, we update action-value functions with Sarsa:

![](_attachments/Screenshot%202022-11-06%20at%2011.27.56.png)

This is particularly effective where states can be reached via multiple paths, since we update can update our states in an online fashion. 
The only thing that has changed vs Monte-Carlo search is the way in which we update $Q(s,a)$. In addition, there is no reason to use table-lookup to represent our $Q(s,a)$. We can use a function approximation instead.

Lastly, let's re-consider the **Dyna** idea. We can combine Dyna with forward search, in an algorithm called **Dyna-2**. 

![](_attachments/Screenshot%202022-11-06%20at%2011.32.38.png)

The key idea is that we maintain two different types of memory. We update one from real experience, and the other form simulated experience. We sum them together to get the overall value function.
The short-term memory is our search-tree. 
