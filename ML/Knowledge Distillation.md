**TL;DR:** Knowledge distillation is the process of transferring knowledge from a larger model to a smaller one.

### Motivation
Large models have high knowledge capacity, but this capacity may not be fully utilised. However, it can be computationally just as expensive to evaluate a model even if it utilises little of its capacity. Smaller models are less expensive to evaluate, so can be deployed on less powerful hardware. 

### Concept 
Transferring knowledge from a large model to a small model needs to somehow teach the latter without loss of validity.
If both models are trained on the same data, the small model may have insufficient capacity to learn a concise knowledge representation given the same computational resources and data as the large model.

However, some information about a concise knowledge representation is encoded in the pseudolikelihoods assigned to its output; the distribution of values among the outputs (for a given input) provides information on how the large model represents knowledge.

Therefore, the training of the smaller model can be achieved by training only the large model on data, exploiting its better ability to learn concise knowledge representations. We then distill such knowledge into the smaller model (that wouldn't be able to learn on its own) by training it to learn the soft output of the larger model.

### Mechanism
Given a large model as a function of $x$ trained for a specific classification task, the final layer of the network is a softmax of the form:

![](_attachments/Screenshot%202022-03-03%20at%2011.26.49.png)

$t$ is a parameter called the *temperature*. For a standard softmax, it is set to $1$. Softmax converts the logits $z_i(x)$ to pseudo-probabilities, and higher values of $t$ create a softer (i.e. flatter) distribution over output classes.

Knowledge distillation consists of training the **distilled** model on a dataset called the **transfer set**. This uses a cross entropy loss between the output of the distilled model $y(x|t)$ and the output $\hat{y}(x|t)$ produced by the large model.

![](_attachments/Screenshot%202022-03-03%20at%2011.31.13.png)

We use a large $t$ for both models. This corresponds to increased entropy of the output. 

The advantages of this distillation mechanism are as follows:

* Key point: this **provides more information for the distilled model to learn compared to hard targets**.
* We can therefore use less data.
* There is also the added benefit that the distilled network trained on soft targets has much less variance in the gradient between training cases.
* We can therefore use a higher LR.



