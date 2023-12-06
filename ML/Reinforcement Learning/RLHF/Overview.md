This series of notes is about [Reinforcement Learning](../Dave%20Silver%20Course/Lecture%201%20-%20Introduction%20to%20RL.md) from Human Feedback (RLHF). They're based upon [this LessWrong post](https://www.lesswrong.com/posts/rQH4gRmPMJyjtMpTn/rlhf), the [Deep RLHF Paper](Deep%20RLHF%20Paper.md), and the [InstructGPT](InstructGPT.md) paper.

## High-Level Summary
RLHF learns a [reward](Lecture%201%20-%20Introduction%20to%20RL#^937012) model for a task based on human feedback. It then trains a policy to optimise the reward received from this model.

RLHF's key advantages are:

* The ease of gathering feedback
* The sample efficiency in training the reward model

For many tasks, it's easier to provide feedback on a model's performance rather than teaching through imitation. We can also imagine tasks where humans remain incapable of completing the task themselves, but can evaluate various completions and provide feedback on them.

The feedback can be as simple as picking the better of two sample completions (as is done in [InstructGPT](InstructGPT.md)), but it's plausible that other forms of feedback might be more effective. The ultimate goal is to get a reward model that represents human preferences for how a task should be done. This is also called [Inverse Reinforcement Learning](Inverse%20Reinforcement%20Learning.md).

