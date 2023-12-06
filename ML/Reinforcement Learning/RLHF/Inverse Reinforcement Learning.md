In traditional [RL](../Dave%20Silver%20Course/Lecture%201%20-%20Introduction%20to%20RL.md), an agent learns to maximise a pre-defined reward function.

[Inverse Reinforcement Learning](https://thegradient.pub/learning-from-humans-what-is-inverse-reinforcement-learning/) aims to learn a reward function from the observed behaviour of some agent (usually an expert).

Consider the example of autonomous driving. Traditional RL would try to specify a reward function that captures the desired behaviour of a driver. But this is not possible in practice - it's just too complex. Instead, Inverse RL takes a set of human-generated driving data and extracts an approximation of the human's reward function for the task.