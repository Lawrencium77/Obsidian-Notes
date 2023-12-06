I tried to get the basics of constitutional AI, to put in the blog.
The paper can be found [here](https://arxiv.org/pdf/2212.08073.pdf).

## The Constitutional AI Approach
The idea is that human supervision comes entirely from a set of AI principles that should govern AI behaviour, along with a small number of examples used for few-shot prompting.
Together, these principles form the constitution.

Their process has two stages:

#### 1. Supervised Stage
* Generate initially harmful responses to a prompt.
* Ask the model to critique its response according to a principle in the constitution, then revise the original response.
* Then finetune a pre-trained LM with supervised learning on the final revised responses.
* The main aim is to get the model "on distribution", i.e. to alter the distribution of responses to reduce the need for exploration in the second RL phase.

It's clear that chain-of-thought style reasoning is pretty key to this approach.

#### 2. RL Stage
This stage mimics RLHF, except we replace human preferences with AI preferences. It could be called RLAIF.

* They distill LM interpretations of a set of principles back into a **hybrid** human/AI reward model.
* They then finetune the model from stage 1 via RL against this reward model, result in a policy trained by RLAIF.