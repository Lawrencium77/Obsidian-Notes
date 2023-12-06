```toc
```

## About RL
How does RL sit within the whole field of science? RL sits at the intersection of many fields of science. It is very general - it's the *science of decision making*.

This crops up in many fields, under many names. For instance, in engineering the study of **optimal control** shares a lot of RL algorithms (with different names).

![](_attachments/Screenshot%202022-09-04%20at%2014.11.11.png)

What makes RL different from other ML algorithms? There are major distinctions:

* There is no supervisor, only a *reward signal*.
* Feedback is delayed, not instantaneous. A bad decision might only have consequences far down the line.
* Time matters - **we are not talking about i.i.d data**. What a robot sees at one second is very correlated to what it will see at the next second.
* Agent's actions affect the subsequent data it receives.

Some fun examples of RL are:

* Flying stunt manoeuvres in a helicopter
* Playing Go
* Controlling a power station
* Managing an investment portfolio

## The RL Problem
### Rewards

^937012

One of the most fundamental quantities in RL is a **reward**. A reward $R_t$ is a scalar feedback signal - it's just a number. $R_t$ refers to the reward signal at time $t$.

It indicates how "well" an agent is doing at step $t$. The agent's job is to maximise cumulative reward. RL is based on the **reward hypothesis** - that **all goals can be described by the maximisation of expected cumulative reward**.

This hypothesis is somewhat controversial. 

In terms of sequential decision making, an agent cannot be greedy. Actions may have long term consequences, and reward may be delayed. It may better to sacrifice immediate reward to gain more long-term reward.

### Environments 
![](_attachments/Screenshot%202022-09-04%20at%2014.26.24.png)

Let's represent an agent with an image of a brain. The goal of RL is to develop algorithms that sit inside this agent. The agent is given observations and rewards at some time step, and uses them to produce an action.

In addition to this, there is also an environment:

![](_attachments/Screenshot%202022-09-04%20at%2014.27.45.png)

We have no influence over the environment, except for though the agents actions. At each step $t$ the agent:

* Executes an action $A_t$
* Receives observation $O_t$
* Receives scalar reward $R_t$

### State
The **history** is the sequence of observations, actions, and rewards:

$H_t = A_1, O_1, R_1, \dots, A_t, O_t, R_t$

i.e. it's all observables variables up to some time $t$. 

What happens next depends on the history (this is the algorithm that we design). But the history isn't usually very useful. It's often too long and unwieldy. Instead, we define a **state**. Loosely, the state is the information used to select actions. 

Formally, it is a function of the history:

$S_t = f(H_t)$

For instance, a valid definition of state would be to look at the last item in the history.

People talk about state in several different ways:

* The **environment state** $S_t^e$ is the environment's private representation. It's the information that's used within the environment to determine what happens next. The environment state isn't usually visible to the agent (e.g. a robot can't know the state of every atom in the Universe). It's more used as a formalism that helps us understand what an environment is.

![](_attachments/Screenshot%202022-09-04%20at%2014.36.04.png)

* What is more useful is the **agent state**. This is essentially just the set of numbers inside our algorithm that's used to decide on future actions. it can be any function of history: $S_t^a = f(H_t)$.

* A more mathematical definition of state is the **information state**, aka **Markov state**. An informations state contains all useful information from the history. This is an information-theoretic concept. We base this on the Markov property:

$P[S_{t+1}|S_t] = P[S_{t+1}|S_1,\dots,S_t]$

This just means that the future is independent of the past, given the present.

The environment state $S_t^e$ is Markov. The history $H_t$ is also Markov (this is maybe a tautology). 

We will now consider a couple of special cases.

A **fully observable environment** is one in which the agent directly observes the environment state:

$O_t=S_t^a=S_t^e$

When we work with this kind of decision, we come up with the main formalism for RL - a **Markov Decision Process (MDP)**.

A **partially observable environment** is one in which the agent *indirectly* observes the environment. For instance, a robot with camera vision isn't told its absolute location.
In this case, $S_t^a \not = S_t^e$.

Formally, this is a **partially observable Markov Decision Process (POMDP)**. An agent must construct its own state representation. For instance, we could use:
* Complete history: $S_t^a=H_t$
* Beliefs of environment state: $S_t^a = (P[S_t^e=s^1],\dots,P[S_t^e=s^n])$
* RNN: $S_t^a = \sigma(S_{t-1}^a + O_tW_o)$

## Inside an RL Agent
Let us now discuss the main components of an RL agent. It may include one or more of these components:

* **Polic**y: the function the agent uses to pick its actions
* **Value function**: function used to evaluate how good each state and/or action is
* **Model**: agent's representation of the environment

Let's go through these one at a time.

A **policy** is the agent's behaviour. It is essentially a mapping from state to action. Our policies can be deterministic ($a=\pi(s)$) or stochastic ($\pi(a|s) = P(A=a|S=s)$).

The **value function** is a prediction of future reward. It is used to evaluate different states. The value function depends on the agent's future behaviour, i.e. the policy:

$v_{\pi}(s) = \mathbb{E}_{\pi}[R_t + \gamma R_{t+1}+\gamma^2R_{t+2}+...|S_t=s]$

The $\gamma$ here is called the **discount factor**. To put it simply: if we follow some particular behaviour, how much reward are we expected to get over time?

A **model** predicts what the environment will do next. We normally have two parts to our model: the **transition model** and **reward model**.
The transition model $\mathcal{P}$ predicts the next state (i.e. the dynamics of the environment).
The reward model $\mathcal{R}$ predicts the next reward.
These can be written like:

$\mathcal{P}^a_{ss'}=P[S'=s'|S=s,A=a]$
$\mathcal{R}^a_{s}=\mathbb{E}[R|S=s,A=a]$

It's worth noting that it's only **optional** to build a model. A lot of this course will focus on **model-free** methods.

Now that we have seen these three quantities, we can categorise RL by which components our agent contains:

* Value Based
	* No policy (this is implicit; we can just choose our action to maximise our value function)
	* Has a value function
* Policy Based:
	* Has a policy
	* No value function
* Actor Critic:
	* Has a policy
	* Has a value function

Perhaps *the* fundamental distinction in RL is:

* Model free
	* Policy and/or value function
	* No model
* Model Based
	* Policy and/or value function
	* Model

A summary of this taxonomy is shown below:

![](_attachments/Screenshot%202022-09-04%20at%2015.18.43.png)

## Problems within RL
Let's begin with learning and planning. There are two fundamental problems in sequential decision making:

1. The Reinforcement Learning problem:
	* The environment is initially unknown
	* The agent interacts with the environment
	* The agent improves its policy
	
2. The Planning Problem:
	* A model of the environment is fully known
	* The agent performs computations with its model (without any external interaction)
	* The agent improves its policy

Of course, one way to do RL is to first learn about the environment, and then do planning. So these two problems are strongly linked.

Another problem within RL is how to balance **exploration and exploitation**.

RL is like trial-and-error learning. The agent should discover a good policy from its experiences of the environment. But it must do so without losing too much reward along the way.

In other words, exploration might mean choosing to find more information about the environment, at the cost of immediate reward.

The final distinction in RL is between **Prediction** and **Control**. Prediction asks us to evaluate future reward, given a policy. Control asks us to find a policy that maximises future reward. 
In RL, we typically need to solve the prediction problem in order to solve the control problem.







