[NCCL Collectives](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/collectives.html?highlight=collective) refer to the set of NCCL operations that enable communication between GPUs. These notes summarise each of the primary NCCL collective operations.

### AllReduce
An [AllReduce](Ring-Based%20AllReduce.md) performs reductions on data across devices and writes the results to every rank.
For example, performing an sum AllReduce across four ranks:

![](_attachments/Screenshot%202023-04-23%20at%2018.50.34.png)

Note that a [Reduce](#Reduce) followed by a [Broadcast](#Broadcast) is the same as an AllReduce.
As is a [ReduceScatter](#ReduceScatter) followed by an [AllGather](#AllGather) (this is the same as what's described in my notes on [Ring-Based AllReduce](Ring-Based%20AllReduce.md)).

### Broadcast
The broadcast operation (c.f. [Broadcasting](PyTorch/Broadcasting.md)) copies an $N$-element tensor from one rank (called "root") to all ranks:

![](_attachments/Screenshot%202023-04-23%20at%2018.51.33.png)

### Reduce
This is the same as an [AllReduce](#AllReduce), except it writes the result to only one device (again, called "root"):

![](_attachments/Screenshot%202023-04-23%20at%2018.52.28.png)

### AllGather
The AllGather operation gathers $N$ values from $k$ ranks into an output of size $k\times N$. It then distributes that result to all ranks:

![](_attachments/Screenshot%202023-04-23%20at%2018.53.40.png)

### ReduceScatter
The ReduceScatter operations performs the same operation as the [Reduce](#Reduce) operation, except the result is scattered in equal blocks between ranks. Each rank gets a chunk of data based on its rank index:

![](_attachments/Screenshot%202023-04-23%20at%2018.54.58.png)

