```toc
```

The next series of lectures focus on scaling up the algorithms that we have previously discussed. The key topic of this lecture is using function approximators to represent the value function.

## Introduction

RL can be used to solve *large* problems. For instance:

* Backgammon: $10^{20}$ states
* Computer Go: $10^{170}$ states
* Helicopter: continuous state space

How can we scale up the model-free methods for *prediction* and *control* from the last two lectures?

So far, we have represented value functions by a **lookup table**. Every state $s$ has an entry $V(s)$. Or every state-action pair has an entry $Q(s,a)$.
But for problems with large MDPs, there are too many states and/or actions to store in memory, and it is too slow to learn the value of each state individually.

The solution for large MDPs is the **estimate the value function with function approximation**:

![](_attachments/Screenshot%202022-10-16%20at%2012.13.42.png)

The aim is to try and build some function that estimates $v_\pi(s)$ or $q_\pi(s,a)$ across the **whole state(-action) space**. I.e. we use some compact representation, where our function approximator has fewer parameters than there are states in the state space.

This also allows us to generalise to states we have not seen.

We update the parameters of our model, $\boldsymbol{w}$, using MC or TD learning.

There are multiple types of value function approximation we can do:

![](_attachments/Screenshot%202022-10-16%20at%2012.18.56.png)

When we do action-value function approximation, there are a couple of different choices. We can either provide $s$ and $a$ and ask it to estimate $q(s,a)$, or we can just provide an $s$, and ask it to evaluate $q(s,a)$ for all possible future actions.

There are many function approximators that we can use:
* Linear combinations of features (we use these in this lecture)
* Neural networks
* Decision trees
* etc...

For RL (in this class, at least) we consider **differentiable** function approximators. Furthermore, we require a training method that is suitable for **non-stationary, non-iid** data (as our data arrives, it arrives in trajectories).  

## Incremental Methods

### Gradient Descent
Just to make the notation clear:

![](_attachments/Screenshot%202022-10-16%20at%2012.24.32.png)

*If we had an oracle* $v_\pi(s)$, then we could simply do supervised learning:

![](_attachments/Screenshot%202022-10-16%20at%2012.26.09.png)

### Linear Function Approximation
Let's get concrete about how we actually do this. This can be done with linear function approximation using **features**. We define a **feature vector**:

![](_attachments/Screenshot%202022-10-16%20at%2012.27.20.png)

Each feature is just some statistic about our state space. E.g. it could be distance of a robot from some landmarks. Or trends in the stock market. If we choose our features well, it compresses information about our state and makes the learning problem easier.

The features chosen are problem-dependent, and are the focus of **feature engineering**. To be clear, this technique is in contrast to **deep RL**, which uses a neural net instead of linear function approximation. 

If we assume we have a good feature, we can make use of them with some **linear combination**:

![](_attachments/Screenshot%202022-10-16%20at%2012.31.53.png)

This means that the objective function is quadratic in parameters $\boldsymbol{w}$: 

![](_attachments/Screenshot%202022-10-16%20at%2012.32.38.png)

Again, this assumes we have some oracle $v_\pi(s)$. SGD converges on a global optimum. The update rule is very simple, as the gradient is simply the feature vector:

![](_attachments/Screenshot%202022-10-16%20at%2012.34.06.png)

#### Table Lookup Features
Let's relate this to our previous lectures. Table lookup is a special case of value function approximation. Using table lookup features:

![](_attachments/Screenshot%202022-10-16%20at%2012.36.29.png)

This means that the parameter vector $\boldsymbol{w}$ gives value of each individual state:

![](_attachments/Screenshot%202022-10-16%20at%2012.37.18.png)

### Incremental Prediction Algorithms
So far, we have cheated! We have assumed an oracle to tell us the "true" $v_\pi(s)$. But in RL, there is no supervisor, only rewards. In practice, we substitute a **target** for $v_\pi(s)$:

![](_attachments/Screenshot%202022-10-16%20at%2012.39.42.png)

Put simply, we substitute in some estimate of $v_\pi$ when updating our weights. 
We can frame this in a way that looks like supervised learning:

![](_attachments/Screenshot%202022-10-16%20at%2012.44.00.png)

We are essentially training our function approximator $v_\pi(s)$ to fit the observed returns. This will converge to a local optimum.
We can do a similar thing for TD(0):

![](_attachments/Screenshot%202022-10-16%20at%2012.46.23.png)

Again, we are building a kind of "dataset" from our TD-targets. Despite the fact that the TD target is biased, it can be shown that linear TD(0) converges close to a global optimum.

> [!INFO]
> There is a subtlety in that we take gradients w.r.t $\hat{v}(S,\boldsymbol{w})$ but not $\hat{v}(S',\boldsymbol{w})$. I don't fully understand it. It's discussed more in the lecture (45:44).
> 

### Incremental Control Algorithms
Let's now move on to the problem of control.

We still use our generalised policy iteration framework, but we use approximate policy evaluation:

![](_attachments/Screenshot%202022-10-23%20at%2012.56.02.png)

We begin with some parameter vector, $\boldsymbol{w}$. Each time we sample an episode, we update our parameters. We can no longer guarantee that $q_{w} \to q_*$ - we can't even guarantee that we can represent $q_*$. But in practice, we can get very close.

The first thing to consider is how to do *action*-value function approximation. We follow a similar process to that used for $v_\pi$.

![](_attachments/Screenshot%202022-10-23%20at%2013.00.35.png)

We can represent our function approximator using some linear approximation, where we represent the state and action by a feature vector:

![](_attachments/Screenshot%202022-10-23%20at%2013.01.27.png)

and then:

![](_attachments/Screenshot%202022-10-23%20at%2013.01.38.png)

We can plug in the same idea for to do incremental control:

![](_attachments/Screenshot%202022-10-23%20at%2013.02.18.png)

#### Example Problem: Mountain Car
This is probably the most frequently used example in all of RL. The setup is that we have a car which is in a dip. It doesn't have enough power to escape:

![](_attachments/Screenshot%202022-10-23%20at%2013.04.14.png)

To get out, the car has to oscillate in order to build up enough momentum to escape. 

Our action space is discrete: at each point in time, we can either accelerate or not.
The state space is 2D: it has a position axis, and a velocity axis. The value function might look like:

![](_attachments/Screenshot%202022-10-23%20at%2013.05.31.png)

This diagram shows the value function as we follow **Sarsa** for model-free control. 

### Convergence
Is it sound to bootstrap? Monte-Carlo is a noisy oracle, but do TD methods always converge?

The answer is no. TD isn't guaranteed to be a stable algorithm. It's important to know when it's safe to use TD, and when it is not:

![](_attachments/Screenshot%202022-10-23%20at%2013.25.41.png)

The right-hand column is related to non-linear function approximation. Note that the combinations which don't *always* work (indicated by an X) can work in many cases; it's just not guaranteed that they always do.

The convergence of Q-learning is as follows:

![](_attachments/Screenshot%202022-10-23%20at%2013.27.39.png)

Control is surprisingly problematic; the generalised policy iteration framework rarely guarantees convergence.  We often end up chattering around the optimal policy.

## Batch Methods
Gradient descent is simple and appealing. But it **not** sample efficient. We see our experience once; after making an update, we then discard it.
The idea of batch methods is to find the best fitting value function to *all* of the data we have seen in our batch. 
In this case, the "batch" is our agent's experience.

### Least Squares Prediction
What does it mean to find the "best fit"? One possible definition is the least-squares fit:

![](_attachments/Screenshot%202022-10-23%20at%2013.32.12.png)

Similar to before, we are constructing a "dataset" by consulting an oracle.

There is a really easy way to find the least-squares solution. This approach is **experience replay**:

![](_attachments/Screenshot%202022-10-23%20at%2013.34.56.png)

Instead of throwing away our data, we store it. We keep our entire dataset in memory. This is just like supervised learning: we make a dataset, randomly sample from it, and update our parameters accordingly.

This converges to the least-squares solution:

![](_attachments/Screenshot%202022-10-23%20at%2013.37.27.png)

As an example, let's look at Experience Replay in Deep Q-Networks (DQN):

![](_attachments/Screenshot%202022-10-23%20at%2013.38.37.png)

We use some CNN to represent our Q-values.
This method is stable with neural networks. There are two tricks that make this stable, compared to naïve Q-learning:

1. Experience replay - this de-correlates trajectories, yielding more stable updates.
2. We use two different Q-networks - we freeze the old network, and bootstrap towards our frozen targets (instead of our latest updates). After (e.g.) a few-thousand steps, we then switch the frozen & active networks.

This was the exact method that they used to develop an AI that can play Atari (all the way back in 2015...).

#### Using Linear Function Approximation
Experience replay finds the least squares solution. But it takes many iterations.

Using *linear* function approximation $\hat{v}(s,\boldsymbol{w})=x(s)^T\boldsymbol{w}$ means we can solve the least squares solution directly.

![](_attachments/Screenshot%202022-10-23%20at%2013.48.33.png)

Again, we consider some oracle. The idea is simple: at the minimum, we do not change our weights. This gives us a closed-form solution for our weights.
The cost of the matrix inversion is a function of the size of our feature vector.

Sometimes, this can be better than experience replay; for instance, if we have  a small amount of experience, and a small number of features.

### Least Squares Control
In practice, this is applied in the generalised policy iteration framework:

![](_attachments/Screenshot%202022-10-23%20at%2013.53.22.png)

> [!INFO]
> There are more details in the lecture slides on this technique. But David skipped over them in the lecture.


