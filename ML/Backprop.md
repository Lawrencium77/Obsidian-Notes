This is a topic that I already understand well. But [Chris Olah's blog post](https://colah.github.io/posts/2015-08-Backprop/) is so neat that I had to make a few notes on it!



## A Reminder
Beyond its use in deep learning, backpropagation is a powerful computational tool in many other areas. In fact, the algorithm has been reinvented at least dozens of times in different fields. The general, application independent, name is **reverse-mode differentiation**.

Consider the following computational graph:

![](_attachments/Screenshot%202023-07-21%20at%2015.24.34.png)

To calculate derivates:

![](_attachments/Screenshot%202023-07-21%20at%2015.25.05.png)

The general rule is to sum over all possible paths from one node to the other, multiplying the derivatives on each edge of the path together. This is just another way of thinking about the multivariate chain rule.

## Factoring Paths
The problem with just “summing over the paths” is that it’s very easy to get a combinatorial explosion in the number of possible paths.

![](_attachments/Screenshot%202023-07-21%20at%2015.26.22.png)

In the above diagram, there are three paths from $X$ to $Y$, and a further three paths from $Y$ to $Z$. If we want to get the derivative $\partial Z / \partial X$ by summing over all paths, we need to sum over $3\times3=9$ paths:

$$\frac{\partial Z}{\partial X}=\alpha \delta+\alpha \epsilon+\alpha \zeta+\beta \delta+\beta \epsilon+\beta \zeta+\gamma \delta+\gamma \epsilon+\gamma \zeta$$

The above only has nine paths, but it would be easy to have the number of paths to grow exponentially as the graph becomes more complicated.

Instead of just naively summing over the paths, it would be much better to factor them:

$$\frac{\partial \mathrm{Z}}{\partial \mathrm{X}}=(\alpha+\beta+\gamma)(\delta+\epsilon+\xi)$$

This is where **forward-mode** **reverse-mode** differentiation come in.
They’re algorithms for efficiently computing the sum by factoring the paths. Instead of summing over all of the paths explicitly, they compute the same sum more efficiently by merging paths back together at every node. In fact, both algorithms touch each edge exactly once.

Forward-mode differentiation starts at an input to the graph and moves towards the end. Reverse-mode differentiation starts at an output of the graph and moves towards the beginning.

Forward-mode differentiation tracks how one input affects every node. Reverse-mode differentiation tracks how every node affects one output.

## Computational Victories
What's the advantage of using reverse mode differentiation in neural nets? Why not use forward-mode differentiation instead?

The answer is simple: forward-mode differentiation gives us the derivative of every output node with respect to one input. Reverse-mode gives us the derivative of every input node with respect to a single output.

Since neural nets have many inputs, and one output, it makes sense to use reverse-mode differentiation. If we used forward mode, we'd have to do a forwards pass for every input variable.

## Isn't This Trivial?
I'm quoting from Chris verbatim here:

> When I first understood what backpropagation was, my reaction was: “Oh, that’s just the chain rule! How did it take us so long to figure out?” I’m not the only one who’s had that reaction. It’s true that if you ask “is there a smart way to calculate derivatives in feedforward neural networks?” the answer isn’t that difficult.
> 
> But I think it was much more difficult than it might seem. You see, at the time backpropagation was invented, people weren’t very focused on the feedforward neural networks that we study. It also wasn’t obvious that derivatives were the right way to train them. Those are only obvious once you realize you can quickly calculate derivatives. There was a circular dependency.
> 
> Worse, it would be very easy to write off any piece of the circular dependency as impossible on casual thought. Training neural networks with derivatives? Surely you’d just get stuck in local minima. And obviously it would be expensive to compute all those derivatives. It’s only because we know this approach works that we don’t immediately start listing reasons it’s likely not to.
> 
> That’s the benefit of hindsight. Once you’ve framed the question, the hardest work is already done.

My take on this is that people didn't realise that gradient descent would work with neural nets (due to local minima). So they never bothered to think about how to calculate gradients.


