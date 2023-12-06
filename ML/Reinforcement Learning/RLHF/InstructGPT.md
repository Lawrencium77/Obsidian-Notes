These notes describe the [InstructGPT paper](https://cdn.openai.com/papers/Training_language_models_to_follow_instructions_with_human_feedback.pdf). They're part of a broader series I made about [RLHF](Overview.md). I found [this Youtube video](https://www.youtube.com/watch?v=VPRSBzXzavo) to be useful.
```toc
```

## Motivation
InstructGPT aims to align language models with user intent, by fine-tuning with human feedback. Predicting the next token on some webpage is different from the objective "follow users' instructions helpfully and safely."

## Method
### High-Level Summary
They train multiple model sizes, all of which use the [GPT-3](../../Transformers/GPT-3.md) architecture.
Their method consists of three steps:

1. Supervised fine-tuning (SFT) on human-written demonstrations.
2. Training a reward model (RM) on comparison data, to predict preferred outputs.
3. Use the RM as a reward function to fine-tune the supervised baseline.

Steps 2 and 3 can be iterated continuously; more comparison data is collected on the current best policy[^fn1], which is used to train a new RM and then a new policy.
Here's a diagram of the method:

![](_attachments/Screenshot%202023-02-18%20at%2017.41.39.png)

To be clear: the SFT model additionally serves as a baseline against which to compare the RLHF model.

### Supervised Fine-Tuning
They fine-tune GPT-3 on labeler demonstrations. They use supervised learning.

### Reward Modeling
They start with the SFT model with the final unembedding layer removed. They train it to take in a prompt and response, and output a scalar reward.

In previous papers, the RM was trained on a dataset of comparisons between two model outputs on the same input. This is a binary classification task, and a cross entropy loss is used.

In InstructGPT, the present labelers with anywhere between $K=4$ and $K=9$ responses to rank. This produces ${K \choose 2}$ comparisons for each prompt shown to a labeler. The loss function used for the reward model is then:

$$L(\theta)=-\frac{1}{{K \choose 2}}\mathbb{E}[\log(\sigma(r_\theta(x,y_w)-r_\theta(x,y_l)))]$$

where $r_\theta(x,y)$ is the scalar output of the reward model for prompt $x$ and completion $y$ with parameters $\theta$. $y_w$ is the preferred completion out of the pair $(y_w, y_l)$.

The intuition is simple: they're training to maximise $r_\theta(x,y_w)-r_\theta(x,y_l)$.

#### Comparing this Loss Function to the Deep RLHF paper

^3d31a1

In the [Deep RLHF Paper](Deep%20RLHF%20Paper.md), the rewards are interpreted in a slightly different way.
They convert rewards to probabilities like so:

$$P[\sigma^1 \succ \sigma^2]=\frac{\exp \sum \hat{r}(o^1_t,a^1_t)}{\exp \sum \hat{r}(o^1_t,a^1_t)+\exp \sum \hat{r}(o^2_t,a^2_t)}$$

where $\sigma^i$ are trajectories. They then **also use a cross entropy loss**:

$$\textrm{loss}(\hat{r})=-(\mu(1)\log P[\sigma^1\succ\sigma^2]+\mu(2)\log P[\sigma^1\prec\sigma^2])$$

It seems the key difference is in **mapping rewards to probabilities**. The InstructGPT paper basically does:

$$P[\sigma^1 \succ \sigma^2]=\sigma(r^1-r^2)$$

whereas the Deep RLHF paper does:

$$P[\sigma^1 \succ \sigma^2]=\frac{\exp(r^1)}{\exp(r^1)+\exp(r^2)}=\frac{1}{1+\exp(r^2/r^1)}=\sigma({r^1/r^2})$$

where $(r^1, r^2)$ are the rewards associated with trajectories $\sigma^1, \sigma^2$ respectively.

### Reinforcement Learning
At this stage, our **language model serves as a policy**. The sequence of tokens it emits serve as actions. And the sequence of text it is conditioned upon serves as the state:

![|500](_attachments/Screenshot%202023-02-18%20at%2018.12.28.png)

We now have a standard policy-based RL problem, with a policy that's specified by some parameters $\theta$. 

Proximal Policy Optimization (PPO) is the RL algorithm used to improve the policy. I won't go in to the details here. Suffice to say that PPO is a [policy gradient method](../Policy%20Gradient%20Methods%20(Sutton%20&%20Barto).md) developed by OpenAI in 2017. It has been popular across several domains.

To prevent overfitting to the reward model, they also add a KL-divergence term to the loss function that penalises differences from the SFT model. 


## Findings
They provide their findings in great detail in the paper. Here, I noted down the interesting parts:

* Labelers prefer InstructGPT outputs over GPT-3
* InstructGPT shows generalisation to instructions outside of the RLHF fine-tuning distribution. They find it able to follow instructions for summarising code, and sometimes follow instructions in different languages. This suggests the models are able to generalise the notion of "following instructions".



[^fn1]: In other words, the new LM is used to generate more responses for preference ranking.