These notes are largely based upon [this paper](https://arxiv.org/pdf/2103.13630.pdf) (which summarises quantization for neural network inference), and a [PyTorch blog](https://pytorch.org/blog/quantization-in-practice/) that is based on the same paper. Additionally, Lei Mao has a [blog post](https://leimao.github.io/article/Neural-Networks-Quantization/) on the topic; the PyTorch docs have a [summary page](https://pytorch.org/docs/stable/quantization.html#model-preparation-for-eager-mode-static-quantization), as well as tutorials on [dynamic](https://pytorch.org/tutorials/recipes/recipes/dynamic_quantization.html) and [static](https://pytorch.org/tutorials/advanced/static_quantization_tutorial.html#post-training-static-quantization) quantisation.

I later found another [paper](https://arxiv.org/pdf/2004.09602.pdf) that gives a great summary. I haven't yet managed to update my notes to reflect its contents.

The aim is to give a brief summary of approaches to quantisation in Neural Networks, particularly for inference.

> [!INFO]
> The original article has a section of *Advanced Concepts*. I did not make any notes on this, since it doesn't seem immediately relevant.
```toc
```

## Section I - Introduction
As soon as computers were used to do abstract mathematical computations, the problem of efficient representation of numerical values in those computations arose. Quantisation is strongly related to this: how should a set of continuous real values be distributed over a discrete set of numbers to minimise number of bits required, and to maximise the accuracy of computation?

An important success of NN quantisation has been at training (think [autocast](https://pytorch.org/docs/stable/amp.html)). However, these notes focus on quantisation at *inference* since most research has focused on this area, and it has proven difficult to go below FP16 at training without significant tuning.

> [!INFO]
> The reason that we often quantise to Int8 is that hardware vendors explicitly allow for faster processing of 8-bit data (than 32-bit data), resulting in higher throughput.

## Section II - General History of Quantisation
Quantisation, as a method to map input values in a large (often continuous) set to output values in a small (often finite) set, has a long history. Rounding and truncation are examples.
Recently, it has been important in digital signal processing, as the process of representing a signal in digital form ordinarily involves rounding. It's also used in numerical analysis and the implementation of numerical algorithms, where computations on real-valued numbers are implemented with finite-precision arithmetic.

The effects of quantisation where first presented in Shannon's paper on the mathematical theory of communication. In particular, he argued in his lossless coding theory that using the same number of bits is wasteful when events of interest have a non-uniform probability. A more optimal approach would be to vary the number the number of bits based on the probability of an event. This is called **variable-rate quantisation**.

Quantisation appears in a slightly different way in algorithms that use numerical approximations for problems involving continuous mathematical properties. Certain algorithms that solve a problem "exactly" in some idealised sense perform very poorly in the presence of noise introduced by roundoff & truncation errors.
(To be specific: roundoff errors are to do with using floating-point arithmetic; truncation errors arise since only a finite number of iterations of an iterative algorithm can be performed.)

### Quantisation in Neural Nets
NN applications are somewhat different; there is not a single problem that is being solved. Instead, we are interested in some sort of forward error metric. Due to over-parametrisation, there are many very different models that optimise this metric. Thus, it is possible to have a high "distance" between a quantised model and its original, while retaining good generalisation performance.

This observation has been the motivation for novel techniques for NN quantisation. The layered structure of NN models also adds an additional dimension to explore. Different layers have different impact on the loss function, which motivates a mixed-precision approach.

## Section III - Basic Concepts

### A. Problem Setup and Notation
Let's assume a NN has $L$ layers with learnable parameters ${W_1,W_2,\dots,W_L}$, with $\theta$ denoting the combination of all such parameters. We focus on the supervised learning problem, where the goal is to minimise:

![](_attachments/Screenshot%202022-10-19%20at%2014.57.55.png)

where $(x,y)$ are the input data and corresponding label. We also denote the input hidden activations of the $i$-th layer as $h_i$, and the corresponding output hidden activation as $a_i$. 

In quantisation, the goal is to minimise the precision of both the parameters, $\theta$, and the intermediate activation maps, $h_i$ and $a_i$, with minimal impact on the generalisation power of the model.

### B. Uniform Quantisation
We need first to define a function that can quantise NN weights and activations to a finite set of values. This function takes real values in floating point, and maps them to a lower precision range, as illustrated in Figure 1: ^9b8db6

![](_attachments/Screenshot%202022-10-19%20at%2015.01.32.png)

I think this is generally quite a nice picture of what quantisation is doing, and it helps to think about concepts in the rest of these notes in its context.

A popular choice of **quantisation function** (aka **mapping function**) is as follows:

![](_attachments/Screenshot%202022-10-21%20at%2010.39.59.png)

where $Q$ is the quantisation operator, $r$ is a real-valued input (activation or weight), $S$ is a real-valued scale factor, and $Z$ is an integer zero point. The $\textrm{Int}$ function maps a real value to an integer value through a rounding operation. In essence this function is a mapping from real values to integers. This method is also called **uniform quantisation**, as the resulting quantised values (aka **quantisation levels**) are uniformly spaced (left of Figure 1).

I really think this equation makes quantisation seem more complex than it actually is. We are just assigning ranges of values to an integer value. $S$ determines how big each of these ranges are, and $Z$ determines which exact number they are matched to. $Z$ is pretty much used as a bias to ensure that a $0$ in floating point maps to a $0$ in the quantised space.

There are also **non-uniform quantisation** methods, whose quantised values are not necessarily uniformly spaced (right of Figure 1). These are discussed further in [Section III.F](#^0cf11d).

We can recover real values $r$ from the quantised values $Q(r)$ through **de-quantisation**:

![](_attachments/Screenshot%202022-10-21%20at%2011.13.36.png)

$\tilde{r}$ will not exactly match $r$,  due to the rounding operation. Their difference constitutes the **quantisation error**.

### C. Symmetric and Asymmetric Quantisation
One important factor in uniform quantisation is the choice of $S$. This scaling factor essentially divides a given range of real values into a number of partitions. It can be defined as:

![](_attachments/Screenshot%202022-10-21%20at%2011.15.34.png)

$b$ is the quantisation bit width.  $[\alpha, \beta]$ denotes the **clipping range**: a bounded range that we are clipping the real values with. A straightforward choice is to use the min/max of the signal for the clipping range, i.e. $\alpha=r_{\min}, \beta=r_{\max}$. This is an example **asymmetric quantisation** (aka **affine quantisation**) since the clipping range isn't necessarily symmetric w.r.t the origin (another example is in Figure 2, right).
The process of choosing the clipping range is known as **calibration**. 

To be clear: the reason the denominator is $2^b-1$ is just *fences and fenceposts*: if we define have 8-bit quantisation, then we define numbers 1-256. This corresponds to a range of $256 - 1 = 255$.

It's possible to use **symmetric quantisation** by choosing a symmetric clipping range, $\alpha=-\beta$. A popular choice is to base these on the min/max values of the signal: $\alpha=-\beta=\max(|r_{\max}|,|r_{\min}|)$. In other words, we just take the most extreme value in our signal, and use this as the clipping value.

These ideas are illustrated in this diagram:

![](_attachments/Screenshot%202022-10-21%20at%2011.22.18.png)

Asymmetric quantisation often results in a tighter clipping range as compared to symmetric quantisation. In addition, it's important to use asymmetric quantisation when the target weights/activations are imbalanced, e.g. the activation after ReLU always has non-negative values.
This is illustrated in the following diagram:

![](_attachments/Screenshot%202022-10-21%20at%2016.02.56.png)

Using symmetric quantisation, however, does simplify the quantisation function:

![](_attachments/Screenshot%202022-10-21%20at%2011.23.52.png)

Here, there are two choices for the scaling factor. In "full range" symmetric quantisation $S$ is chosen as $\frac{2\max{|r|}}{2^n-1}$, to use the full INT8 range of [-128,127]. We can also use $\frac{\max{|r|}}{2^{n-1}-1}$, which only uses [-127,127]. 
Symmetric quantisation is widely adopted in practice for quantising weights because ensuring the $0$ in floating point maps to $0$ in the quantised output can significantly reduce computation during inference.

Using the min/max of the signal for both symmetric and asymmetric quantisation is a popular method. However, it can be susceptible to outlier data int the activations. These could unnecessarily increase the range and, as a result, reduce the resolution of quantisation. One approach to address this is to use the **percentile** instead of min/max of the signal. 
Another approach is to select $\alpha$ and $\beta$ to minimise KL divergence between the real values and quantised values.

**Summary (Symmetric vs Asymmetric Quantisation)**
Symmetric quantisation partitions the clipping using a symmetric range. This has an easier implementation, as it sets $Z=0$. However, it is sub-optimal for cases where the range could be skewed and not symmetric. For such cases, asymmetric quantisation is preferred.
In practice, symmetric quantisation is often used for weights, and affine quantisation is used for activations.

### D. Range Calibration Algorithms: Static vs Dynamic Quantisation
So far, we have discussed approaches to determining the clipping range $[\alpha, \beta]$. Another important factor is **when** the clipping range is determined. 

It can be computed statically for weights, as in most cases the parameters are fixed during inference. However, the activations differ for each input. As such, there are two approaches to quantising activations: **static**, and **dynamic** quantisation.

In dynamic quantisation, the range is **dynamically** calculated for each set of activations during runtime. In other words, activations are quantized "on-the-fly" during inference. 
This requires real-time computation of the signal statistics (min, max, percentile, etc) which can have a very high overhead. However, it does result in higher accuracy as the signal range is exactly calculated for each input.

Another approach is static quantisation, in which the clipping range is pre-calculated and **static** during inference. This does not add any computational overhead, but it typically results in lower accuracy as compared to dynamic quantisation. One popular approach is to run a series of calibration inputs to compute the typical range of activations. See the right of [Figure 4](#^a44b37) for an illustration.
Multiple metrics have been proposed to find the best range, including minimising MSE between the original unquantised weight distribution and the corresponding quantised values.

**Summary (Dynamic vs Static Quantisation)**
Dynamic quantisation computes the clipping range of each activation and often achieves highest accuracy. However, calculating the range of a signal dynamically is very expensive. As such, static quantisation is often used (it also avoids the float <-> int conversion cost between layers). But it's worth noting that dynamic quantisation is preferred for models like Transformers where writing/retrieving the model's parameters from memory dominate bandwidths.

> [!Question]
> My read is that a clipping range is computed for each activation individually. Is this correct?

### E. Quantisation Granularity
This section was based upon a different source (i.e. [this paper](https://arxiv.org/pdf/2004.09602.pdf)).

There are several choices for sharing quantisation parameters among tensor elements. This choice is known as **quantisation granularity**:

* At the coarsest, **per-tensor granularity** means that the same quantisation parameters are shared by all elements in the tensor.
* The finest granularity would have individual quantisation parameters per element.
* Intermediate granularities reuse parameters over various dimensions of the tensor - per row or per column for 2D matrices, per channel for 3D (image-like) tensors, etc.

There are two factors to consider when choosing granularity: model accuracy, and computational cost. For activations, only per-tensor is practical for performance reasons. Weights should be quantised at either per-tensor or per-column granularity for linear layers of the form $Y=XW$.

### F. Non-Uniform Quantisation
^0cf11d
Some work has explored non-uniform quantisation, where **quantisation steps and quantisation levels** can be non-uniformly spaced. The formal definition is as follows:

![](_attachments/Screenshot%202022-10-21%20at%2012.10.28.png)

$X_i$ represents the discrete quantisation levels and $\Delta_i$ the quantisation steps (thresholds). 
Specifically, when the value of a real number $r$ falls in between $\Delta_i$ and $\Delta_{i+1}$, the quantizer $Q$ projects it to the corresponding quantization level $X_i$.

Non-uniform quantization may achieve higher accuracy for a fixed bit-width, since one could better capture the distributions by focusing more on important value regions.  For instance, many non-uniform quantisation methods have been designed for Gaussian distributions of the weights/activations that often involve long-tails.

**Summary (Uniform vs Non-Uniform Quantisation)**
Generally, non-uniform quantisation enables us to better capture signal information. However, non-uniform quantisation schemes are difficult to deploy efficiently on general computation hardware like GPU/CPU. As such, uniform quantisation is the current de-facto method of quantisation.

> [!INFO]
> There is more detail on non-uniform quantisation methods in the paper. But I skipped it, mainly because non-uniform quantisation isn't very common.

### G. Fine-Tuning Methods
It's often necessary to adjust the parameters of a NN after quantisation. This can either be performed by re-training the model - **Quantisation Aware Training (QAT)** - or done without re-training - **Post-Training Quantisation (PTQ)**. 
A schematic comparison between these two approaches is show in Figure 4: ^a44b37

![](_attachments/Screenshot%202022-10-21%20at%2014.55.15.png)

**Quantisation-Aware Training**
Given a trained model, quantisation may introduce a perturbation to the trained model parameters. This can push the model away from the point to which it had converged when training with floating point precision. This can be addressed by re-training the model with quantised parameters. 

In **Quantisation-Aware Training** the usual forward and backward pass are performed on the quantised model in floating point, but the model parameters are quantised after each gradient update. An illustration is shown in Figure 5:

![](_attachments/Screenshot%202022-10-21%20at%2015.20.22.png)

My intuition for this procedure is as follows: we store our parameters in floating point, but during the forwards pass we simulate the quantisation operation. This ensures we adjust the parameters such that they produce useful output after quantisation.

One subtlety is how to backprop through the (non-differentiable) quantisation operator. Without any approximation, its gradient is almost zero everywhere (see [Figure 1](#^9b8db6)). A popular solution is to approximate the gradient of this operator with the so-called **Straight Through Estimator (STE)**. This essentially ignores the rounding operation, and approximates it with an identity function. Despite the coarse approximation, STE works well in practice.

In PyTorch, this is implemented as follows: all weights and biases are stored in FP32, and backprop happens as usual. However, in the forwards pass, quantisation is internally simulated by quantising and immediately de-quantising the parameters. This adds quantisation noise similar to that encountered during quantised inference.

QAT tends to converge to flatter minima than models that are quantised with PTQ:

![](_attachments/Screenshot%202022-10-21%20at%2016.11.02.png)

**Summary (QAT)**
QAT has been shown to work despite the coarse approximation of STE. However, the main disadvantage of QAT is the computational cost of re-training. It may need to be performed for a significant period of time. This is only likely to be worth it if the model is going to be deployed for an extended period.

**Post-Training Quantisation**
An alternative to the expensive QAT is Post-Training Quantisation. This performs quantisation without any fine-tuning. As such, the overhead is often negligible but often lower accuracy.

### H. Stochastic Quantisation
During inference, the quantisation scheme is usually deterministic. However, this is not the only possibility. Some works have explored **stochastic quantisation for quantisation aware training**, as well as reduced-precision training.
The high-level intuition is that stochastic quantisation may allow a NN to explore more. One popular supporting argument has been that small weight updates may not lead to any weight change, as the rounding operation may always return the same weights. But stochastic rounding may provide the NN an opportunity to **escape**, thereby updating its parameters.

More formally, stochastic quantisation maps the floating-point number up or down with a probability related to the magnitude of the weight update:

![](_attachments/Screenshot%202022-10-21%20at%2012.20.30.png)

However, a major challenge with stochastic quantisation methods is the overhead of creating random numbers for every single weight update. **As such, they are not yet adopted widely in practice.**

### Extra: Sensitivity Analysis
This section is based on the PyTorch blog, rather than the paper.

Not all layers respond to quantisation equally. Some are more sensitive to precision drops than others. Identifying the optimal combination of layers than minimises accuracy can be expensive; it can be effective to do a **one-at-a-time** sensitivity analysis to identify which layers are the most sensitive, and then retain FP32 precision in those.

In PyTorch, this looks something like:

![](_attachments/Screenshot%202022-10-21%20at%2016.30.51.png)

Another approach is to compare statistics of FP32 and INT8 layers; commonly used metrics for these are **Signal to Quantised Noise Ratio (SQNR)**, and MSE. Schematically, this looks something like:

![](_attachments/Screenshot%202022-10-21%20at%2016.32.13.png)

## Suggested Workflow
The following diagram is a suggested workflow, from the PyTorch blog. It also serves as a handy summary of the most useful techniques discussed in these notes.

This ignores [SmoothQuant](../../Speechmatics/quantization/SmoothQuant.md), which I know to be very useful.

![](_attachments/Screenshot%202022-10-21%20at%2016.37.21.png)

Note that, in all cases, we quantise both the weights **and** activations of the model. 

Lei Mao's blog has a handy summary of the three main quantisation methods:

![](_attachments/Screenshot%202022-10-21%20at%2020.35.13.png)

> [!Question]
> How does the accuracy of dynamic quantisation compare to a) Sensitivity Analysis + Selective PTQ; and b) QAT?

> [!Question]
> When we do QAT, do we then do PTQ afterwards? Or can we do dynamic quantisation?

