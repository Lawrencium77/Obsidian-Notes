An argument that can be passed to [DDP](DDP.md) is `gradient_as_bucket_view`. From the [PyTorch docs](https://pytorch.org/docs/stable/generated/torch.nn.parallel.DistributedDataParallel.html#torch.nn.parallel.DistributedDataParallel):

```python
torch.nn.parallel.DistributedDataParallel(module, device_ids=None, ..., gradient_as_bucket_view=False, ...)
```

What is the point of this flag?

Usually, PyTorch stores model gradients on device. DDP also creates additional buffers in the form of communication buckets. 
These buckets are created when we instantiate our DDP model (with `DDP(model)`). They store a copy of the gradients which are used for the [AllReduce](../../Ring-Based%20AllReduce.md). 

When we set `gradient_as_bucket_view=True`, it ensures that all gradients will instead be [views](../Reshapes%20vs%20Views.md) pointing to different offsets of DDP buckets. 

This reduces peak memory usage by an amount be equal to the total gradients size. Moreover, it avoids the overhead of copying between gradients and `allreduce` communication buckets.

