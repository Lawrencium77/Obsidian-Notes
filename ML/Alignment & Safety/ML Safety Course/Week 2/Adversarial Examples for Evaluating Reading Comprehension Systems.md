These notes summarise [this paper](https://arxiv.org/pdf/1707.07328.pdf), which I did for my Week 2 reading.

> [!INFO]
> This paper was published in 2017, so is out of date on several ways.

```toc
```

## Introduction
* The extent to which LMs truly understand language is unclear.
* They propose adversarial evaluation for NLP. 
* They focus on the [SQuAD](https://rajpurkar.github.io/SQuAD-explorer/) reading comprehension task.
* They SoTA models they had at the time achieved 84.7% F1 score. They can get this down to much lower with their technique.
* Prior work in CV adds noise to images, relying on the fact that it won't change the true label. In contrast, changing just one word can alter a paragraph's meaning.
* So they create adversarial examples by adding distracting sentences to the input paragraph. For instance:

![](_attachments/Screenshot%202023-03-05%20at%2012.28.47.png)

## The SQuAD Task and Models
* The SQuAD dataset (original version) has > 100k human-generated reading comprehension questions about paragraphs from Wikipedia articles.
* They test an LSTM and and attention-flow model. They use a single & ensemble version for each $\to$ 4 models in total.

## Adversarial Evaluation

### General Framework
* They define an adversary $A$ to be a function that takes an example $(p,q,a)$ (paragraph, question, answer) with a model $f$ and returns a new example $(p',q',a')$.
* For this to be useful, $(p',q',a')$ must be **valid** (a human would think them consistent), and they should somehow be "close" to $(p,q,a)$.

### Concatenative Adversaries
* One method of building adversaries is to generate samples of the form $(p+s,q,a)$ for some sentence $s$. In other words, we add a new sentence to the end of the paragraph, and leave the question and answer unchanged. 
* Valid adversarial examples are those for which $s$ doesn't contradict the correct answer.
* They have two main types of concatenative adversaries: AddSent and AddAny:

![](_attachments/Screenshot%202023-03-05%20at%2012.42.56.png)

* In AddSent, we generate a new Q & A using a bunch of hand-written rules. They then use the new Q & A to generate a new statement to append to the example. They check that this new sentence makes grammatical sense by crowdsourcing.
* For AddAny, the goal is to choose any sequence of $d$ words, regardless of grammatical correctness. They deliberately choose distracting examples, by measuring how much the model's output probability distribution changes.

## Experiments
* They measure F1 adversarial score across 1000 random samples from SQuAD.
* AddSent made F1 score drop from 76% to 31%, averaged across all models.
* AddAny made it drop to 7%.
* They verified that humans aren't fooled by their examples. Performance dropped by 13.1 F1 points - much less than the AI.
* They tried training on adversarial examples. Since AddSent and AddAny are computationally expensive, they instead run only steps 1 - 3 of AddSent to generate an adversarial sentence for each training example. They then train BiDAF from scratch.
* They find that the adversarially trained model does well on their specific adversarial scheme, but still fails on other types of adversarial example. The models seems to overfit the adversary.




## Discussion
* They claim that to optimise adversarial evaluation metrics, we may need new strategies for training models. For certain classes of models and adversaries, efficient training strategies do exist.
* E.g. adversarial training can be used for any model trained with SGD, but it requires generating adversarial examples at every iteration. This is feasible for images, but is infeasible for domains where only slower adversaries are available.
* While we use adversaries as a way to evaluate language understanding, robustness to adversarial attacks may also be its own goal for tasks such as spam detection.
* They relate their work to other papers that proposed harder test datasets for various tasks. E.g. LAMBADA tests the ability of LMs to handle long-range dependencies.
* They finish by remarking that building truly intelligent systems is only possible if our evaluation metrics can distinguish real intelligent behaviour from shallow pattern matching.





