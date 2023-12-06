These notes describe the [deep RLHF paper](https://arxiv.org/pdf/1706.03741.pdf). They're part of a broader series I made about [RLHF](Overview.md). 

```toc
```

## Motivation
To train RL agents in real-world environments, we need to communicate complex goals. Suppose we wanted an RL that scrambles eggs - how could we construct a reward function?

As a solution, the authors explore goals defined by human preferences. They do so in the context of Atari games and simulated robot movement.

## High-Level Summary
They train a **reward model** to predict human preferences while simultaneously training a **policy** to optimise the predicted reward function.

Here is a schematic:

![|400](_attachments/Screenshot%202023-02-18%20at%2018.42.40.png)

## Method

### Preliminaries
Consider an agent interacting with an environment over a sequence of steps.
In traditional RL, the environment would supply a reward $r_t$. Instead, we assume there is a human overseer who can express preferences between **trajectory segments**.

A trajectory segment is a sequence of observations and actions. Informally, the goal of the agent is to produce trajectories which are preferred by the human, while making as few queries as possible to the human.

### Overview
At each point in time, we maintain a policy $\pi$ and reward model $\hat{r}$. 
Both models are updated by three processes:

1. The policy $\pi$ interacts with the environment to produce a set of trajectories $\{\tau^1, \dots, \tau^i\}$. The parameters of $\pi$ are updated by a traditional RL algorithm.
2. Select pairs of segments from the trajectories produced in step 1 to send to a human for comparison.
3. There are used to update the reward model $\hat{r}$, with supervised learning.

These processes run asynchronously, with:

* **Trajectories** flowing from 1 $\to$ 2
* **Human comparisons** flowing from 2 $\to$ 3
* **Parameters** for $\hat{r}$ flowing from 3 $\to$ 1

These steps are exactly the same as those in the [InstructGPT](InstructGPT.md) paper.

### Policy Optimisation
To solve policy optimisation, we can use any appropriate RL algorithm. We focus on [policy gradient methods](../Policy%20Gradient%20Methods%20(Sutton%20&%20Barto).md). Specifically, [advantage actor critic](https://arxiv.org/abs/1602.01783) for Atari and [TRPO](https://arxiv.org/abs/1502.05477) for robotic tasks. I assume that a modern version of this paper would use [PPO](https://openai.com/blog/openai-baselines-ppo/), as in [InstructGPT](InstructGPT.md).

### Fitting the Reward Function
We can interpret a reward function estimate $\hat{r}$ as a preference predictor. We assume the human's probability of preferring a segment $\sigma^i$ depends exponentially on the normalised sum of the reward:

$$P[\sigma^1 \succ \sigma^2]=\frac{\exp \sum \hat{r}(o^1_t,a^1_t)}{\exp \sum \hat{r}(o^1_t,a^1_t)+\exp \sum \hat{r}(o^2_t,a^2_t)}$$

This mapping from rewards to probabilities is essentially a sigmoid of the ratio of rewards, i.e. $\sigma(r^1/r^2)$. My [InstructGPT notes](InstructGPT#^3d31a1)  describe this in more detail.

We choose $\hat{r}$ to minimise the cross entropy loss between these predictions and the human labels:

$$\textrm{loss}(\hat{r})=-(\mu(1)\log P[\sigma^1\succ\sigma^2]+\mu(2)\log P[\sigma^1\prec\sigma^2])$$
where $\mu$ is the (one-hot) ground truth distribution. 

The actual algorithm has some modifications to this approach. An important difference is that they use an **ensemble** of predictors. The estimate $\hat{r}$ is defined by independently normalising each of these predictors, and averaging the result.

#### Selecting Queries
We decide which trajectories to send to humans for labeling based upon the uncertainty in the reward function estimator. Those with the highest variance across ensemble members are sent for evaluation.





