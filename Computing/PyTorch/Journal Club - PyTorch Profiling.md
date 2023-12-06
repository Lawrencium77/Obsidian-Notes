Use something like:

```Python
profiler = torch.profiler.profile()
profile.start()
```

If you have a distributed job, it will run the profiler on all tabs.

It is compatible with tensorboard.

Shows you, for each of your workers, how much time is spent waiting, doing communication. Shows you a step time breakdown: how much time is spent executing CUDA kernels, doing a memcopy, doing stuff on the CPU, etc.

# Hidden State Cache
When accumulating gradients, we can do it in two different ways.

One is to accumulate within a stream.

![](_attachments/Screenshot%202022-08-09%20at%2016.51.17.png)

We take consecutive windows from the same stream within each batch. This harms convergence, as there is less variance within each batch.

Instead, we can accumulate between streams:

![](_attachments/Screenshot%202022-08-09%20at%2016.52.01.png)

Since the TXL is stateful, we need to make sure this state is managed in the correct way.