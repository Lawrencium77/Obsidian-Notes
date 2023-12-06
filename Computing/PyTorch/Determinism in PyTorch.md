These notes are based on [this](https://pytorch.org/docs/stable/notes/randomness.html#reproducibility) page in the PyTorch docs, and [this](https://pytorch-dev-podcast.simplecast.com/episodes/torchuse-deterministic-algorithms-7OUBuJKe) PyTorch Dev Podcast.

Completely reproducible results are not guaranteed in PyTorch. Only *some* PyTorch ops are non-deterministic; being deterministic is sometimes quite expensive, and allowing for some non-determinism can give big speedups.

One obvious way to control reproducibility is to set manual seeds for all random number generators.

### Avoiding nondeterministic algorithms
The function `torch.use_deterministic_algorithms` sets whether PyTorch operations must use deterministic algorithms. When enabled, operations will use deterministic algorithms when available. If only nondeterministic algorithms are available, they throw a `RuntimeError`.

It's important to understand that **individual kernels can exhibit nondeterminism**. This nondeterminism often originates from backing libraries. For example, CUDNN's convolution implementation is nondeterministic.

A common reason that an individual kernel is nondeterministic is down to the [non-associativity of floating point arithmetic](../CSAPP/Chapter%202/Floating%20Point.md). When parallel operations concurrently accumulate into a buffer, the order of additions can vary, leading to nondeterminism. In CUDA, most uses of atomic add are nondeterministic.

