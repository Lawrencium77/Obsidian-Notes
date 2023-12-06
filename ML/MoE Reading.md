```toc
```

## Wed 12th

### Review Paper

[GShard](#GShard) and the [Switch Transformer](#Switch%20Transformer) replace FF layers with expert layers. But other work revisits the concept of experts as fully independent models $\implies$ benefit of modularity and composability.

Generic sparse neural nets (with irregular sparsity patterns) reduce overall theoretical FLOPs but are not efficiently supported on current hardware, which specialise in linear algebra on contiguous blocks of memory.

Early architectures employed many, small experts that'd fit within an individual accelerator. But later works had experts be split across accelerators.

The notion of *effective parameter count* seems important - how is it calculated? It's explained further in the Deepmind paper.

The quality of sparse models is disproportionately reduced under domain shift, or when fine-tuning on different task distributions. This is especially true on reasoning-heavy tasks.

There are three common categories to choosing which experts should get which tokens:

1. Each token chooses top-$k$ experts
2. Each expert chooses top-$k$ tokens
3. Globally determine which token should go to which expert

People have applied MoE to the ViT.

On a *per-parameter* basis, sparse models will always look comparatively worse to dense models.

Models with a high number of experts do well under pre-training but the best performing models under transfer have used larger, fewer experts.

Prior work routes at the *task* level instead of the word or token level for machine translation. Their reasoning is that this allows for more efficient inference, since only a subset of weights is required.

Prior work has tracked the most frequent prior input token when an expert was selected. They find specialisation in quantities, numbers, and clusters of related verbs, nouns, etc.

Other work has done a similar thing with a multi-model text-image model.

### Switch Transformer
JAX code and all models checkpoints are available. 

During [Knowledge Distillation](Knowledge%20Distillation.md), they find that initialising the dense model with non-expert weights gives a modest improvement.

Switch [Transformer](Transformers/Attention%20is%20All%20You%20Need.md) translates substantial upstream gains better to knowledge-based tasks than to reasoning tasks. Specifically, for a fixed upstream perplexity, they find that both Switch and dense models perform similarly in the small to medium model size regime. But in the largest model regime, Switch models don't always translate their upstream perplexity well to downstream fine-tuning on the SuperGLUE task (which is reasoning-heavy).

During training, the router determines the best expert based on the incoming activation vector. However, it receives no counterfactual information about how well it would have done selecting an alternate expert. As in RL, a classic [exploration-exploitation](Reinforcement%20Learning/Dave%20Silver%20Course/Lecture%209%20-%20Exploration%20and%20Exploitation.md) problem arises. Deterministically selecting the top expert always amount to an exploitative strategy - they consider a few different ways to incorporate exploration.

The one they settle on is to multiply the incoming representation by some Gaussian noise. They found it empirically perform well. This is also done in [Outrageously Large Neural Networks The Sparsely-Gated Mixture-of-Experts Layer (Noam Shazeer)](#Outrageously%20Large%20Neural%20Networks%20The%20Sparsely-Gated%20Mixture-of-Experts%20Layer%20(Noam%20Shazeer)).

> [!INFO]
> Note that the Switch Transformer is an encoder-decoder model.

### MoE with Expert Choice Routing
Instead of letting tokens select the top-$k$ experts, they have experts select the top-$k$ tokens. This guarantees perfect load balancing, and allows a variable number of experts for each token.

MoE is applied to the dense FF layer because this is the most computationally expensive.

They train several variants of their architecture with an expert size of 100M parameters.

To summarise their method: for an input $X\in\mathbb{R}^{B\times L\times d}$ (where $B$ and $L$ are batch size and sequence length), they flatten into a tensor of shape $X\in\mathbb{R}^{N\times d}$, where $N=B\times L$. They then let each expert pick their top-$k$ tokens.

But this means they're completely screwed for autoregressive text generation: their method requires taking in past and future tokens to perform the top-$k$ selection. One solution is to collect a large batch of input sequences, split tokens with the same sequence index into separate groups, and perform expert choice routing for each group.
But they're still screwed when batch size becomes very small.

### Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer (Noam Shazeer)
The first approach to using MoE in large neural nets.
They insert an MoE layer between two LSTM layers. They got good results on machine translation.
Uses a top-$k$ routing function.
Use tensor parallelism, and do some communication efficiency optimisations.

### GShard
Not too much to add tbh.

### Beyond Distillation: Task-Level Mixture of Experts for Efficient Inference
All this work is in the context of machine translation (just using a typical encoder-decoder architecture).

They test routing strategies at the token-, sentence-, and task-levels. Tasks are defined either by the target language, or the language pair.

They're motivated by inference efficiency: if you only need a subset of experts for a task, then you can basically prune your model, thereby keeping it all on one GPU.

How they decide which experts should receive which task isn't totally clear; it looks like they inject some task information into the token embeddings, and then use a basic matmul. I think the authors are deliberately obfuscating their technique.

They claim that prior work shows that distillation has been found to "introduce undesirable artefacts" and only retains "a small fraction of the gains achieved by scaling".

Using sentence representations to route examples performs badly.

They examine routing decisions in **token-level** MoE models, distinguishing by task. For instance:

![](_attachments/Screenshot%202023-07-12%20at%2016.27.02.png)

## Thu 13th

### Branch-Train-Merge: Embarrassingly Parallel Training of Expert Language Models
In the review paper, they say this work composes expert language models trained on specific domains, e.g scientific or legal text. It's "embarrassingly parallel" in that each expert is a fully formed LM. 

They use the term ELMFOREST to refer to a set of EXPERT LMs (ELMs), each specialised on a distinct domain. They have no shared parameters.

The algorithm they use for this is called Branch-Train-Merge (BTM). To create a new ELM:

> 1. Initialise a new LM with an average of the parameters of the most relevant LMs in the current set.
> 2. Train on new data
> 3. Add into the current set of ELMs

BTM is initialised with a single LM that's trained of heterogeneous data.

At inference, they do one of two things:

#### Ensembling
Essentially does a weighted sum of the output of the most relevant ELMs. To estimate $p(X_t|\boldsymbol{x}_{<t})$:

$$p(X_t|\boldsymbol{x}_{<t})=\sum_{j=1}^kp(X_t|\boldsymbol{x}_{<t},D=j)\cdot p(D=j|\boldsymbol{x}_{<t})$$
where we use the top-$k$ experts according to their domain posterior. The domain posterior is calculated using Bayes' rule:

$$
p\left(D=j \mid \boldsymbol{x}_{<t}\right)=\frac{p\left(\boldsymbol{x}_{<t} \mid D=j\right) \cdot p(D=j)}{p\left(\boldsymbol{x}_{<t}\right)}=\frac{p\left(\boldsymbol{x}_{<t} \mid D=j\right) \cdot p(D=j)}{\sum_{j^{\prime}=1}^k p\left(\boldsymbol{x}_{<t} \mid D=j^{\prime}\right) \cdot p\left(D=j^{\prime}\right)}
$$

Note that this procedure requires a fwd pass through $k$ ELMs.

#### Model Averaging
They use parameter averaging to collage the ELMFOREST into a single LM. They use the same weighted average as above.

> [!IDEA]
> The distinction between a *task* and *domain* seems important for us. Here, all experts are defined by their domain. 


### DEMix Layers: Disentangling Domains for Modular Language Modelling
By the same authors as [above](#Branch-Train-Merge%20Embarrassingly%20Parallel%20Training%20of%20Expert%20Language%20Models). But came first.

According to the review paper: DEMix explicitly has different experts for different pre-training domains. Experts can then be selected by doing domain matching on the inputs.

The idea is pretty simple. We use a standard MoE architecture (replacing FF blocks), but at training we explicitly reserve one expert for each task. In other words, the gating function is:

$$g_j\left(\mathbf{h}_{t, \ell}\right)= \begin{cases}1 & \text { if } j=d \\ 0 & \text { otherwise }\end{cases}$$

where $d$ is the domain label for the current training instance, $\boldsymbol{h}_{t,l}$. This means they use 1 expert per domain. Their set of 8 domains is very basic, with examples being 'Computer Science' and 'Legal'.

At inference, text may not come with a domain label or even belong to a single domain. So they use exactly the same trick as in the above [Ensembling](#Ensembling) section. In other words, they take a weighted average across experts.

They point out that they can train new experts very easily, without having to update the other experts or shared parameters.

### Lifting the Curse of Multilinguality by Pre-Training Modular Transformers
Not too much in this paper. The key point is that they use a MoE approach in a multilingual model; each expert corresponds to a language:

![|500](_attachments/Screenshot%202023-07-13%20at%2011.48.38.png)

### DeepSpeed-MoE: Advancing Mixture-of-Experts Inference and Training to Power Next-Generation AI Scale
They being by noting that the scope of past MoE models in NLP is primarily limited to encoder-decoder architecture. Applications to auto-regressive models is less explored. So they explore this themselves.

They use a couple of new techniques: **Pyramid-Residual MoE** and **Mixture-of-Students**.

The key intuition behind the **PR-MoE** architecture is twofold:

1. Experiments show that placing experts in the later layers improves model accuracy much more than placing them in earlier layers. This suggests the later layers learn more specialized representations that benefit more from experts.
    
2. Using a residual connection where each token passes through both a fixed MLP and an expert MLP provides accuracy improvements similar to using two experts per token. This shows the experts help correct representations from the fixed MLP.

Based on these observations, the PR-MoE model:

- Uses more experts in later layers of the network.
    
- Replaces some layers with a residual design where each token passes through a fixed MLP and expert MLP. The expert acts like a correction module. They argue this means they get the benefit of using 2 experts per layer with the same amount of communication as a Top-1 gating function.

The architecture looks like:

![](_attachments/Screenshot%202023-07-13%20at%2014.16.43.png)

Note that the last two layers have twice as many experts.

For **knowledge distillation**, they distill from a large MoE model into a smaller MoE model. This is unlike previous work, which studies distillation into dense models. They call this **Mixture of Students**. 
Their intuition is as follows:

1. Naive knowledge distillation hurts accuracy when distilling an already compact PR-MoE model into a smaller student.
2. The cause is that the student's capacity is insufficient to minimise both the original training loss and the distillation loss jointly. The student ends up under-fitting as it tries to optimise both losses.
3. A solution is to use staged distillation - teach the student using distillation initially when its capacity is very low, then stop and only optimise the original loss once its capacity catches up.
4. This staged distillation allows the student to leverage the teacher's knowledge when its capacity is limited. Then once capable enough, it fine-tunes solely on the original task without the distillation loss.

The core intuition is that staged distillation sidesteps the negative effects of student underfitting by only using distillation when beneficial. The student first relies on the teacher, then once capable enough switches to self-learning. Retaining the MoE structure also preserves sparse activation benefits.

The last part of interest to me was how they do **inference**. There is a paradox in serving MoE models:

* From the best-case view, each input token of an MoE model only activates a single expert at each MoE layer. 
* From the worst-case view, the aggregate parameters needed to process a batch of tokens can be as large as the full model size.

They implement some engineering solutions to help. One of these is quite interesting - they group and route all tokens with the same critical data path together. This reduces data access per device.

> [!QUESTION]
> Does this mean have to know the critical path of each token before doing a forward pass? One possibility is that they do this with an initial lightweight gating-only forward pass to predict expert assignments and create a mapping of tokens to experts. 
> Or maybe I'm totally misinterpreting them.

### Unified Scaling Laws for Routed Language Models
This studies the scaling properties of sparse expert models across different model sizes and number of experts.

They find that routed models follow a bilinear scaling law in base model size $N$[^fn1] and number of experts $E$:

$$\log L(N,E)=a\log N + b\log E + c\log N \log E + d$$

Note that this describes scaling behaviour *at convergence*. In other words, data is not a bottleneck.

Using their scaling laws, they come up with the notion of **effective parameter count (EPC)**. EPC maps the performance of a routed model with base size $N$ and $E$ experts to an equivalent dense model size that achieves the same loss.
Specifically, if a routed model has validation loss $L(N,E)$, its EPC is defined as the $N'$ that satisfies:

$$L(N', 1) = L(N, E)$$

where $L(N', 1)$ is the loss a dense model with $N'$ parameters. 

Another observation they make is that downstream task performance also benefits from routing and base model scaling, but task-specific analyses show variation in the relative improvements.

### Efficient Large Scale Language Modelling with Mixtures of Experts
This is a Meta paper. The main reason I read it is that their models have been released in the [Fairseq repo](https://github.com/facebookresearch/fairseq/tree/main/examples/moe_lm).

Their models are GPT-style and roughly match the sizes and architectures used in the GPT-3 paper. They train MoE models that mirror their dense models, such that comparisons are approximately matched in terms of the number of FLOPs.

This table summarises their models:

![](_attachments/Screenshot%202023-07-13%20at%2016.15.02.png)

They use alternating dense and expert layers, and a top-2 expert selection. They use 512 experts in each expert layer.

Interestingly, they report not needing to change model initialisation as in the [Switch Transformer](#Switch%20Transformer) paper. Instead, the rescale observe that expert parameters have an $E$-times smaller batch size relative to dense parameters, so rescale expert gradients by $\frac{1}{\sqrt E}$.

They accept that a limitation of their work is that their study is limited to one specific MoE configuration.

## Fri 14th
### Towards Understanding Mixture of Experts in Deep Learning

They claim to be the first people to attempt a theoretical analysis of MoE models. 

They wonder why it is that all experts initially have the same structure (are initialised from the same distribution), yet diverge to different functions. Especially since the router is initialised uniformly.

They look at a toy classification problem with intrinsic cluster structures. They train a **single MoE layer, where each expert is a 2-layer CNN** . So they're **not looking at transformers**.

They prove that any single expert can't achieve an accuracy of > 87.5%. And that the problem can't be solved by a mixture of *linear* experts. So having multiple experts, each of which is non-linear, is crucial.

They also prove that each expert will be specialised to a specific portion of the data, determined by the weight initialisation. The router can learn cluster-centre features and route the input data to the correct experts.

> [!NOTE]
> I don't actually understand their data distribution. But I don't think it's too important.


[^fn1]: I.e. the number of parameters that an individual token interacts with.