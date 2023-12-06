The reading for this week was [this paper](https://arxiv.org/pdf/1706.06083.pdf).

```toc
```

## Introduction
* They motivate robustness from the view that classifiers are being used in production environments. So resistance to adversarial attack is a "crucial design goal".
* How can we train deep neural nets that are robust to adversarial inputs?
* Previous work doesn't offer a good understanding of the *guarantees* that attack/defence mechanisms provide.
* This paper studies adversarial robustness via "robust optimization". They use a saddle point formulation to formalise the notion of adversarial robustness.
* They then summarise their contributions.

## An Optimization View on Adversarial Robustness
Consider a standard classification task. We've an underlying data distribution $\mathcal{D}$ over pairs of examples $x\in\mathbb{R}^d$ and corresponding labels $y\in[k]$.Our goal is to find model parameters that minimise $\mathbb{E}_{(x,y)\sim\mathcal{D}}L(x,y,\theta)$.

To train models that are robust to adversarial attack, we need to augment this paradigm. We choose to propose a concrete **guarantee** that an adversarially robust model should satisfy.

We obtain the following saddle point problem, which is our central object of study:

![](_attachments/Screenshot%202023-03-12%20at%2011.36.32.png)

This is the composition of an inner maximisation problem (defence), and and outer minimisation problem (attack). The rest of this paper investigates the structure of this problem.


## Towards Universally Robust Networks
How to find a solution to this equation?

We now discuss an experimental exploration of the non-concave inner problem.

### The Landscape of Adversarial Examples
They experimentally investigate the local maxima for multiple models on MNIST and CIFAR10. Their main tool is [PGD](Lecture%20-%20Adversarial%20Robustness#Projected%20Gradient%20Descent%20(PGD)). To explore a large part of the loss landscape, they re-start PGD from different initialisations.

They find that the inner problem is tractable. While there are many local maxima, they tend to have very well-concentrated loss values. This points towards PGD being a "universal" adversary.

### First-Order Adversaries
This suggests that robustness against PGD yields robustness against all first-order adversaries.

### Descent Directions for Adversarial Training
Their experiments show that by applying SGD using the gradient of the loss at adversarial examples, we can consistently reduce the loss of the saddle point problem during training.


## Network Capacity and Adversarial Robustness
We need to also show that the final loss we achieve is actually a low number. This corresponds to our classifier actually being good.

At a high level, classifying examples in a robust way requires a stronger classifier, since the presence of adversarial examples changes the decision boundary of a problem to a more complex one.

Their experiments verify that capacity is crucial for robustness. Specifically, they run experiments for CNNs on MNIST and find:

* Larger models are more robust, without adversarial training.
* Increasing capacity reduces the effectiveness of transferred adversarial inputs.


## Experiments: Adversarially Robust Deep Learning Models
Previous experiments demonstrate that we need to a) train a big model and b) use the strongest possible adversary.
The adversary of choice is PGD. They do this for CIFAR10 and MNIST.

They find that their networks are very robust on MNIST, but not on CIFAR10.






