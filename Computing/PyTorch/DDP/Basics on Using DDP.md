These are just a few notes on how [DDP](DDP.md) is actually used. I made them in preparation for my [Conjecture](../../../Random/Applications/Application%20Process/Conjecture.md) interview.


### Basics

To launch a distributed job, one approach is to do something like:

```python
import torch
import torch.multiprocessing as mp

def demo_basic(rank, world_size):
	# code

def run_demo(demo_fn, world_size):
	mp.spawn(demo_fn,
			args=(world_size,),
			nprocs=world_size,
			join=True,
	)

if __name__ == "__main__":
	n_gpus = torch.cuda.device_count()
	run_demo(demo_basic, n_gpus)
```

They key point is that `mp.spawn` launches multiple CPU processes. Each process will run `demo_basic()`, with `rank` as one of its arguments. We can then write our `demo_basic()` function to use only 1 GPU, with index specified by its rank. For instance, we will do something like:

```python
model = model.to(rank)
```

We spawn as many Python processes as there are GPUs. These processes step through the whole train script. However, it's also possible to launch multiple subprocesses associated with the dataloader. This is controlled by the `num_workers` argument.

### Initialisation
For each process group, our `demo_basic()` function will need to call:

```python
def setup(rank, world_size):
	"""Called at the start"""
	os.environ['MASTER_ADDR'] = 'localhost'
	os.environ['MASTER_PORT'] = '12355'
	dist.init_process_group("nccl", rank=rank, world_size=world_size)
```

This function ensures that every process will be able to coordinate through a master, using the same IP address and port. By setting the following four environment variables on all machines, all processes will be able to properly connect to the master:

* `MASTER_PORT`: A free part on the machine that will host the process with rank 0.
* `MASTER_ADDR`: IP address of the machine that will host the process with rank 0.
* `WORLD_SIZE`: Set so that the master knows how many workers to wait for.
* `RANK`

### Collective Communication
I am already fairly [familiar](../../NCCL%20Collectives.md) with these concepts. In PyTorch, the specific meaning of collectives is to allow for communication patterns across all processes in a **group**. The `dist.init_process_group` function creates such a group.

As an example, to obtain the sum of all tensors on all processes, we can use `dist.all_reduce(tensor, op, group)`.

### Datasets
In order to ensure similar results when changing the number of processes, we have to partition any dataset that we use. Here's an example:

```python
class Partition(object):
	"""Helper class, represents data subset"""
    def __init__(self, data, index: List[int]):
        self.data = data
        
        # List of indices that make up partition
        self.index = index

    def __len__(self):
	    # Size of partition
        return len(self.index)

    def __getitem__(self, index):
        data_idx = self.index[index]
        return self.data[data_idx]


class DataPartitioner(object):
	"""Splits dataset into partitions"""
    def __init__(self, data, sizes: List[float], seed=1234):
	    # sizes: list of fractions of how to split dataset
	    
        # Create indexes for whole dataset
        self.data = data
        self.partitions = []
        data_len = len(data)
        indexes = [x for x in range(0, data_len)]
	    
	    rng = Random()
        rng.shuffle(indexes)

        # Store random indices in partitions list
        for frac in sizes:
            part_len = int(frac * data_len)
            self.partitions.append(indexes[0:part_len])
            indexes = indexes[part_len:]

    def use(self, partition):
        return Partition(self.data, self.partitions[partition])

def partition_dataset():
	""" Partitions MNIST"""
    dataset = # MNIST
    
    size = dist.get_world_size()
    bsz = 128 / float(size)
	
	# Equally-sized partitions
    partition_sizes = [1.0 / size for _ in range(size)]
    partition = DataPartitioner(dataset, partition_sizes)
    partition = partition.use(dist.get_rank())

	# Dataloder used to create batches 
    train_set = torch.utils.data.DataLoader(partition,
                                         batch_size=bsz,
                                         shuffle=True)
    return train_set, bsz
```


### Alternatives to `mp.spawn`
An alternative to `mp.spawn` is to use `torch.distributed.launch` or, most appropriately, `torchrun`. These are more suitable for multi-GPU settings, where all GPUs are used on every node.

To use `torch.distributed.launch` you'd do something like:

```
$ python -m torch.distributed.launch --use-env train_script.py
```

but `torchrun` provides a superset of the functionality of `torch.distributed.launch`. It provides elasticity such that the number of nodes can change between a minimum and maximum number. To use it:

```
$ torchrun train_script.py
```

So an example of a single-node multi-worker might look something like:

```
torchrun
    --standalone
    --nnodes=1
    --nproc-per-node=$NUM_TRAINERS
    YOUR_TRAINING_SCRIPT.py (--arg1 ... train script args...)
```








