These notes are based one of David's journal clubs, which was based on [this blog](https://charlesfrye.github.io/programming/2023/11/10/llms-systems.html).

At training, a large batch size and long sequence length collapse to be a single unit of parallelism that we can throw at the GPU. But at inference, we often use an autoregressive decode with a batch size of 1. Increasing batch size comes at the cost of increased latency. The crux of the problem is in achieving good throughput despite this.

**Assisted generation** is one class of approaches that seek to resolve this. In general, these focus on "helping" the model decode using some helper model.

The rest of these notes outline some different papers that have been released in the vein:

* [Speculative Decoding](Speculative%20Decoding.md)
* [Medusa](Medusa.md)
* [Paged Attention](Paged%20Attention.md)