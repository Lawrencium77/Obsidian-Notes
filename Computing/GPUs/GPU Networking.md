I made these notes to cover the high-level details of [GPU](GPUs.md) [networking](../CSAPP/Chapter%2011/Chapter%2011%20-%20Network%20Programming.md). They are based mainly on [this ArXiv paper](https://arxiv.org/pdf/1903.04611.pdf), [this short blog post](https://fuse.wikichip.org/news/1224/a-look-at-nvidias-nvlink-interconnect-and-the-nvswitch/?utm_content=cmp-true), [this Stackoverflow comparing DMA and RDMA](https://stackoverflow.com/questions/34589423/what-is-the-difference-between-rdma-and-dma#:~:text=DMA%20is%20usually%20accessing%20memory,central%20processing%20unit%20(CPU)), and ChatGPT (of course).

Terms covered include:

- Network Topology
- PCIe
- PCIe Switch
- NVLink
- NVSwitch
- NVIDIA DGX
- P2P
- NVMe
- InfiniBand
- Mellanox
- RDMA

## PCIe, NVLink, and NVSwitch
Traditionally, GPU-GPU communication would share the same bus interconnect as CPU-GPU communication. This would always be PCIe.
For context, PCIe can be used to connect CPUs with GPUs, SSDs, network cards, and other high-speed peripherals. Compared to the interconnect between CPU and DRAM, PCIe is much slower.

A diagram of a simple architecture is shown below:

![|250](_attachments/Screenshot%202023-08-05%20at%2011.04.51.png)

Importantly, PCIe **cannot** be used to directly connect GPUs. Instead, a PCIe Switch is required, as shown in the above diagram. 

This was changed by the introduction of the GPU-oriented interconnects, such as **NVLink**. An NVLink can be viewed as a cable with two terminal-plugs, where each GPU incorporates several NVLink slots. How these slots are connected via the NVLink cables determines the **topology** of the GPU network.

Related to this, **NVSwitch** was invented to enable more complex system topologies than can be achieved when using NVLink. In particular, it facilitates all-to-all communication between GPUs, which is super important in neural network training. It is an example of a **network switch**, and is built upon NVLink. Put simply, it's an NVLink Switch.

> [!INFO]
> The concept of a **network switch** (e.g. **PCIe switch**, NVSwitch) is important but not one that I understand. To really get it, I'll need to learn about [networking](Chapter%2011%20-%20Network%20Programming]). Suffice to say that a network switch manages traffic between devices on a network.


To view the topology of a system, in matrix form, we can use this following command:

```
nvidia-smi topo -m
```

**NVIDIA DGX** are a product that contains some number of GPUs (usually 8 or 16) connected by NVLink/NVSwitch.

Finally, I've included this diagram of the topology for each box on the Speechmatics grid:

![](_attachments/Pasted%20image%2020230805112255.png)

This diagram sums up pretty much everything described above. In addition, it raises the concept of **Peer-to-Peer (P2P)**.

In a general sense, [P2P](https://en.wikipedia.org/wiki/Peer-to-peer) communication is exactly what is sounds like: peers are equally privileged participants in a network. They avoid the use of a centralised administrative system. In the context of intra-node GPU communication, P2P means something a little more specific. Consider this diagram:

![](_attachments/Screenshot%202023-08-05%20at%2020.31.53.png)

We are basically doing DMA, except it's between the DRAM of two GPUs, instead of some external device and CPU DRAM. Interestingly, Nvidia disables P2P on its consumer-grade cards, except for via NVLink. This is the reason that B7 supports P2P, whereas B4 does not. Although B7 doesn't support P2P between all cards (i.e., any two that must communicate via the CPU bridge).

#### NVMe
A strongly related concept to P2P is that of DMA between GPU DRAM and secondary storage. This is illustrated by the diagram below:

![](_attachments/Screenshot%202023-08-05%20at%2020.58.53.png)

This approach is made feasible by [NVMe](https://en.wikipedia.org/wiki/NVM_Express), which allows for super quick access to secondary storage. NVMe is a communication protocol specifically developed for SSDs.

Traditionally, HDDs (and early SSDs) used communication protocols that were designed with the limitations of these technologies in mind. For instance, HDDs can only be accessed serially. In contrast, SSDs can access data in parallel. This is a fundamental shift in how data is accessed and these older protocols weren't designed to take full advantage.

NVMe was designed specifically for SSDs, thereby allowing for greater parallelism. NVMe operates over PCIe.

Not all SSDs support NVMe. NVMe compatibility in an SSD is dependent upon:

1. A PCIe connection
2. The SSD controller and firmware

## Inter-Node Communication
Until now, we have only discussed GPU networking within a single node. What technology exists to improve communication between nodes?

One example of such a technology is **InfiniBand (IB)**. IB is a combination of hardware and software components that enable high-speed communication within networks. Example hardware components include:

* Adapters: IB **Host Channel Adapters** connect a computer/device to the IB network.
* **Switches**: IB switches manage the routing of data between devices on a network.
* **Cables**: Special cables are used to physically connect devices within an IB network.

Software components include:

* **Drivers**: Software drivers are used to interface with the IB hardware.
* InfiniBand **APIs**

IB was developed by multiple companies, including Mellanox (now part of NVIDIA), Intel, and IBM. As a compute cluster interconnect, IB competes with **ethernet**, amongst others.

Crucially, IB supports **RDMA**. 
#### DMA and RDMA
In a traditional setup, data transfers between devices and memory have to be managed by the CPU. Hence the need for [DMA](1%20-%20Hardware%20Basics#^7bdae2), which allows a dedicated controller to handle these tasks. 

**RDMA** extends DMA to data transfers between computers over a **network**. It allows one computer in a network to directly access the memory of another computer, without involving either computer's OS.

Nvidia GPUs have the ability to do [GPUDirect-RDMA](https://docs.nvidia.com/cuda/gpudirect-rdma/#:~:text=GPUDirect%20RDMA%20is%20a%20technology,video%20acquisition%20devices%2C%20storage%20adapters.). This enables third-party devices to directly access **GPU DRAM**, without any assistance from CPU or staging through the main memory. This is exactly the same idea as **P2P** (above), except that it operates *between* instead of *within* nodes.

When training neural nets, RDMA is super important. GPU RDMA enables direct memory transfers between the DRAM of multiple GPUs, even when those GPUs are located on different nodes. 









