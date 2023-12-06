These notes are based on [this Wikipedia page](https://en.wikipedia.org/wiki/Automatic_differentiation).

**Automatic Differentiation** (**autodiff**) refers to a set of techniques used to evaluate the derivative of a function specified by a computer program. 

It exploits the fact that every computer program, no matter how complex, executes a sequence of elementary arithmetic operations (addition, subtraction, etc) and elementary functions (sin, cos, etc). By applying the **chain rule** repeatedly to these operations, derivatives of arbitrary order can be computed automatically.

```toc
```

## The Chain Rule
Fundamental to autodiff is the decomposition of differentials provided by the **chain rule**. Consider:

![](_attachments/Screenshot%202022-08-29%20at%2011.14.48.png)

In this context, $x$ is the *independent* variable and $y$ is the *dependent* variable. The chain rule gives:

![](_attachments/Screenshot%202022-08-29%20at%2011.15.04.png)

## Forward Accumulation
There are two distinct modes of autodiff: **forward accumulation** and **reverse accumulation**. Forward accumulation requires we compute the chain rule from inside to outside: we compute $\frac{dw_1}{dx}$, $\frac{dw_2}{dw_1}$, then $\frac{dy}{dw_2}$. Reverse accumulation is traversal from outside to inside: $\frac{dy}{dw_2}$, then $\frac{dw_2}{dw_1}$, then $\frac{dw_1}{dx}$.

For our purposes, we won't focus on forward accumulation as it is not used for backprop. However, it's worth noting that in forward autodiff, the *independent variable* is fixed.

## Reverse Accumulation
In reverse accumulation autodiff, the *dependent variable* is fixed and the derivative is computed w.r.t each sub-expression recursively. In neural nets, the dependent variable is often the loss.

In a pen-and-paper calculation, the derivative of the *outer* function is repeatedly substituted with the chain rule:

![](_attachments/Screenshot%202022-08-29%20at%2011.23.10.png)

The quantity of interest is the **adjoint**, denoted with a bar ($\bar{w}$). It is the derivative of a chosen dependent variable w.r.t a subexpression $w$: ^0f7290

![](_attachments/Screenshot%202022-08-29%20at%2011.24.37.png)

A graphical example of reverse accumulation is shown below:

![500](_attachments/Screenshot%202023-01-29%20at%2012.36.06.png)

