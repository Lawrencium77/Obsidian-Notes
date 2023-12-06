These notes summarise my readings on a [Pipeline Parallel](Engineering%20Challenges%20in%20Training%20Massive%20Transformers#Pipeline%20Parallelism) (PP) implementation in PyTorch.

```toc
```

## Hugging Face Docs
These [Hugging Face Docs](https://huggingface.co/transformers/v4.9.0/parallelism.html) have some useful info.
They first describe **Naive Model Paralllel (NVP)**:

* Groups of model layers are spread across GPUs.
* The mechanism for implementing PP is relatively simply - switch the desired layers `.to()` the desired devices. Whenever the data goes in and out of those layers, switch the data to the same device as the layer and leave the rest unmodified.
* The following diagram shows an example:

![](_attachments/Screenshot%202023-04-27%20at%2013.46.25.png)

* When data needs to travel between GPUs, there is an additional overhead. If the GPUs are on the same node (i.e. physical machine), then this copying is pretty fast. But if the GPUs are on different nodes, the overhead could be larger.
* The main deficiency is that all but one GPU is idle at any moment.

PP solves the GPU idling problem:

* It chunks the incoming batch into [micro-batches](#^3f3c5a) and artificially creates a pipeline. This allows different GPUs to concurrently participate in the computation process.
* The following illustration from the [GPipe paper](https://proceedings.neurips.cc/paper_files/paper/2019/file/093f65e080a295f8076b1c5722a46aa2-Paper.pdf) shows naive MP on top, and PP on the bottom:

![](_attachments/Screenshot%202023-04-27%20at%2013.51.08.png)

* The idle zone is referred to as a **"bubble"**. This occurs since the last `forward` stage has to wait for `backward`.
* It introduces a new hyper-parameter to tune: `chunks`. This defines how many chunks of data are sent in a sequence through the same device. In the above diagram, `chunks=4`.
* Conceptually, this is similar to [Gradient Accumulation](../../ML/Gradient%20Accumulation.md).
* Due to these chunks, PP introduces the concept of **Micro-batches** (MBS). ^3f3c5a
* If you've [Data Parallel](DDP/DDP.md) with degree `dp_degree`, then your global batch size is given by:

```
global_bsz = mbs * chunks * dp_degree
```

* The chunks trade-off is as follows:
	* `chunks = 1` $\implies$ naive MP, which is very inefficient.
	* large `chunks` $\implies$ tiny micro-batches, which is inefficient on GPU.
* So we have to experiment to find the highest GPU utiliization.
* Problems in implementing:
	* Have to modify model quite heavily; we must write the normal flow of modules into an `nn.Sequential`.
	* The Pipeline API is very restricted:
		* Requires either a single Tensor or a tuple of Tensors as the only input and output. These tensors must have `bsz` as the very first dimension.

 The implementations of Pipeline Parallel are:
 * PyTorch
 * FairScale
 * DeepSpeed

Combining DP & PP is as you'd expect. Here's an example from the [DeepSpeed tutorial](https://www.deepspeed.ai/tutorials/pipeline/):

![](_attachments/Screenshot%202023-04-27%20at%2014.07.41.png)

Data parallel ranks are kept completely separate. According to DP, the only GPUs that exist are 0 & 1. GPU-0 "secretly" offloads some of its load to GPU-2 using PP. GPU-1 does similar with GPU-3.


## Tied Weights
One complication with pipeline parallel relates to tied weights. For instance, [transformers](../../ML/Transformers/Attention%20is%20All%20You%20Need.md) commonly use an embedding layer early in the pipeline to map vocabulary to hidden states, and then use the embedding to map hidden states back to vocabulary at the end of the pipeline.

This is obviously problematic for pipeline parallel.

According to [Deepspeed](https://www.deepspeed.ai/tutorials/pipeline/#expressing-pipeline-models), their solution is to replicate tied layers on every pipeline stage that owns an instance of reuse. Training then proceeds as normal, but an additional all-reduce of the tied gradients is added after all backward passes complete. The all-reduce ensures that the weights of the tied layer remain in sync across pipeline stages.


