These notes are about **Generative Pre-Trained [Transformer](Attention%20is%20All%20You%20Need.md) 3 (GPT-3)**. They are derived from the original paper: [Language Models are Few Shot Leaners](https://arxiv.org/pdf/2005.14165.pdf), as well as this super useful [Stanford AI Lab Blog](http://ai.stanford.edu/blog/in-context-learning/).

```toc
```

### Summary
Previous work has demonstrated gains in NLP by pre-training on a large corpus of text, followed by task-specific fine-tuning. The fine-tuning stage typically requires thousands or tens of thousands of examples.

By contrast, humans are few-shot learners. The authors show that scaling up language models greatly improves task-agnostic, few-shot performance.

GPT-3 is an autoregressive language model with 175 Billion parameters. They test it in the few-shot setting: GPT-3 is applied without any gradient updates or fine-tuning. It achieves strong performance in translation, question-answering and [cloze](https://en.wikipedia.org/wiki/Cloze_test) tasks.

GPT-3 can generate samples of news articles which human evaluators have difficulty distinguishing from articles written by humans.

At the same time, they also identify some datasets where GPT-3's few-shot learning still struggles.

### Background on Autoregressive LMs
The Stanford AI blog has a nice section on this.

For autoregressive LMs such as the GPT-`N` family, text generation works by feeding in (aka "conditioning") the model on some input text, transformed beforehand into a sequence of numeric tokens and then one-hot vectors. 

Feeding data as input to the model is called "conditioning" because we can think of model outputs as being a PD that's conditional upon the input, i.e. $p(y|x)$, where $x$ is a vector of inputs.

For an autoregressive LM, the random variable we condition on is the sequence of input text, and the output for the next token is a categorical PD over all possible tokens (~ 50K in the case of GPT-3).

Predictions are later post-processed into strings of text by a predefined mapping that is the inverse of the mapping going from text to tokens used to encode text.

There are different schemes to choose what sequence of output tokens the model predicts, like simple greedy decoding, or nucleus sampling that at each point samples from a re-weighted version of the PD.


### In-Context Learning
Significant progress has been  in NLP by fine-tuning pre-trained transformer language models. As discussed in the [next section](#^3b7b1f), this has many issues.

One approach to addressing these is meta-learning. In the context of LMs, this means the model learns a broad set of skills at training time, and then uses those abilities at inference to adapt to the desired task.

![](_attachments/Screenshot%202022-03-21%20at%2011.50.18.png)

Recent work attempts to do this via **in-context learning**, using the text input of a pretrained language model as a form of task specification: the model is conditioned upon an NLP instruction and/or a few demonstrations of the task and is then expected to complete further instances of the task by predicting what comes next.

A more detailed explanation is as follows: for a given task, the model receives as input an optional task description along with some number of examples demonstrating the task, up until some final example that the model should complete itself. For many tasks that involve taking a query and producing an answer, the examples are pairs of queries and answers formatted in some consistent way.

All this is fed into the LM as a single, continuous chunk of text for the model to continue with generations that it computes as being most probable. Magically, GPT-3 often continues the pattern so that the immediate next tokens it generates turn out to be the answer for the last unfinished query.

When you stop and think about it, *in context learning is remarkable*. Conditioning the model on an example effectively enables the model to adapt on-the-fly to novel tasks whose distributions of inputs vary significantly from the training distribution. The idea that simply minimizing the loss of a single next-word-prediction objective implies this apparent optimization of many more arbitrary tasks is very powerful. We are essentially "learning" during inference time.

It is now clear why in-context learning is a type of meta-learning: the model has *learned to learn* from inputs.

Understanding how and why in-context learning works is an open question. Why is it something that seems to emerge as models get bigger? ^[In this sense, we can view the emergence of in-context learning as a type of phase transition. Are there other behaviours that will arise as models continue to grow?] Would better in-context learning lead naturally to zero-shot learning?

Recent trends have shown that the log loss follows a smooth trend of improvement with scale. Since in-context learning abilities involves absorbing many skills and tasks within model parameters, it's plausible that they might show similarly strong gains with scale. GPT-3 can be seen as a test that validates this hypothesis.

### Testing Regimes
GPT-3 is tested in multiple settings. These can be seen as lying on a spectrum of how much task-specific data they rely on:

**Fine-tuning**: Updating the weights by training on a supervised dataset specific to the desired task. Typically $10^3$-$10^5$ of labeled examples are used. Main advantage: strong performance on many benchmarks. Main disadvantage: the need for a new large dataset for every task, the potential for poor generalisation out-of-distribution, and the potential to exploit spurious features of the training data.
In this work, they don't fine-tune GPT-3 because they focus on task-agnostic performance. ^3b7b1f

**Few-Shot**: The model is given a few demonstrations of the task at inference time as conditioning, but **no weight updates are allowed**. A dataset has a context and desired output, and few-shot works by giving $K$ examples of context and output, and then one final example of context, with the model expected to provide the output. 
Main advantages: reduction in need for task-specific data and reduced potential to learn an overly narrow distribution. Main disadvantage: Results are much worse than SoTA fine-tuned models.

**One-Shot**: Same as few shot, except only 1 demonstration is allowed.

**Zero-shot**: No demonstrations are allowed. The model is only given a natural language instruction describing the task.

The following shows the 4 methods using the example of translating English to French: 
![](_attachments/Screenshot%202022-03-20%20at%2021.38.42.png)

### Implementation
**Model**
They use the same architecture as GPT-2. They train 8 different sizes of the model, ranging over 3 orders of magnitude. The biggest model is what they call GPT-3.

**Dataset**
NLP datasets have rapidly expanded, culminating in the Common Crawl dataset, that has nearly 1 Trillion words. The size of this dataset is sufficient to train their largest models without ever seeing the same sequence twice.

They found that unfiltered versions of Common Crawl tend to have lower quality, so they took a few data processing steps to improve its average quality.

Interestingly, datasets they viewed as higher-quality were sampled more frequently such that some datasets were sampled 2-3 times. This essentially accepts a small amount of overfitting in exchange for higher quality training data.

A big issue with LMs pre-trained on internet data - particularly large models with capacity to memorise vast amounts of content - is potential contamination of downstream tasks by having their test sets inadvertently seen during training. To reduce this contamination, they searched for any overlaps with the test sets of all benchmarks. A bug in the filtering caused them to ignore some overlaps, and the costs of training meant it wasn't feasible to re-train the model.

Section 4 of the paper describes this in detail. 

**Training Process**
Larger models can typically use a larger batch size, but require a smaller LR. To train larger models without running out of memory, they use a mixture of model parallelism within each matrix multiply and model parallelism across the layers of the network.

### Brief Results
The results section of this paper is very lengthy, and not worth summarising in detail. However, they highlight some key results in the introduction.

Firstly, they find that LMs make increasingly efficient use of in-context information as they increase in scale:

![](_attachments/Screenshot%202022-03-21%20at%2012.02.14.png)

Second, they give a rough sense of overall results in the figure below, which aggregates the various tasks the model is tested on:

![](_attachments/Screenshot%202022-03-21%20at%2012.03.03.png)

### Limitations
Text Synthesis: GPT-3 samples still sometimes repeat themselves, start to lose coherence over sufficiently long passages, contradict themselves, and occasionally contain sentences that appear totally unrelated to previous ones (*non-sequitir*).

Discrete Language Tasks: GPT-3 seems to have particular difficulty with common-sense physics.

They focused on exploring in-context learning behaviour with autoregressive models. They don't include any bidirectional architectures or other training objectives, such as denoising. Their design decision comes at the cost of potentially worse performance on tasks that empirically benefit from bidirectionality, such as fill-in-the-blank tasks.

A more fundamental limitation: Scaling up any model may mean we run into the limits of the pre-training objective.

Poor sample efficiency during pre-training. GPT-3 still sees much more text during pre-training than a human sees in a lifetime.

Models at the scale of GPT-3 can be expensive to perform inference on. One possible future direction to address this is knowledge distillation.

### Broader Impacts
The authors have a lengthy section on broader impacts, that makes for pretty interesting reading!

Language models have a wide range of beneficial applications for society, including:
* Code and writing auto-completion;
* Grammar assistance;
* Improving search engine responses;
* Answering questions.

They also have potentially harmful applications. The authors identify 2 categories that these can fall into:

**Misuse of Language Models**: The ability of GPT-3 to generate several paragraphs of synthetic content could easily be misued.

**Bias**: Biases in training data may lead models to generate stereotyped content. Internet-trained models may have internet-scale biases. For instance, they found associations between gender and occupation.

