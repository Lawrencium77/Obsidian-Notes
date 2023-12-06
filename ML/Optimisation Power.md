These notes are based on [this](https://www.lesswrong.com/posts/Q4hLMDrFd8fbteeZ8/measuring-optimization-power) post by Eliezer.

Consider some optimisation agent. Suppose you have:

1. The agent's preference ordering
2. A measure of the space of outcomes

In this case, then we can quantify how "impressive" the agent's optimisation ability is by comparing it to random chance. In other words, we can count the total number of states with equal or greater rank in the preference ordering to the outcome achieved, and divide this by the total space size.

In other words, for some utility $u_\textrm{agent}$, then we measure:

$$\frac{N(u\geq u_\textrm{agent})}{N_\textrm{total}}$$

Note that we don't actually have to be able to *measure* a state's utility. We need only be able to rank states according to our preferences. But I thought it makes the maths easier to express.

Since these probabilities are often tiny (think about how unlikely Shakespeare is given all text sequences of equal length), we take a $\log_2$ and call this **optimisation power in bits**:

$$\textrm{Optimization Power in Bits}=-\log_2 \biggl( \frac{N(u\geq u_\textrm{agent})}{N_\textrm{total}}\biggr)$$

Intuitively, if we find an outcome that is preferable than 50% of all states, then we've done 1 bit of optimisation. 

It's obvious that we are measuring an entropy here. Specifically, we're measuring the amount of "surprise" that we'd expect if we randomly picked a state with equal-or-better score, according to our preference ordering.




