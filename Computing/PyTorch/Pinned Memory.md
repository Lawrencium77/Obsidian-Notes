These notes are based upon several sources, including:
* [Wikipedia](https://en.wikipedia.org/wiki/Virtual_memory#Pinned_pages)
* An [NVIDIA Lecture](https://engineering.purdue.edu/~smidkiff/ece563/NVidiaGPUTeachingToolkit/Mod14DataXfer/Mod14DataXfer.pdf)
* The [PyTorch docs](https://pytorch.org/docs/stable/data.html#memory-pinning)

## What is Pinned Memory?

In general terms:
* **Pinned** memory (aka **page-locked** memory) is a memory area that is never swapped to disk. This idea is general to any operating system.
* Some [pages](VM%20as%20a%20Tool%20for%20Caching#^27a1b0) may be pinned for short periods of time, others may be pinned for long periods of time, and still others may be permanently pinned. 
* Examples of memory that must be pinned include:
	* The paging supervisor code and drivers for secondary storage devices (on which pages reside) must be permanently pinned. Otherwise, paging wouldn't even work because the necessary code wouldn't be available.
	* Data buffers that are accessed by peripheral devices that use [DMA](Processors%20Read%20and%20Interpret%20Instructions%20Stored%20in%20Memory#^515cc8) must reside in pinned pages while the I/O operation is in progress because such devices expect to find data buffers located at physical memory addresses; transfers cannot be stopped if a page fault occurs and then restarted when the page fault has been processed.

This second example (DMA) is very important for CUDA programming:

![](_attachments/Screenshot%202023-06-27%20at%2017.56.38.png)

> [!INFO]
> DMA uses a system interconnect, typically a PCIe (for NVIDIA hardware).
* An address is translated and page presence is checked for the entire source and destination regions at the beginning of each DMA transfer.
* There is then no address translation for the rest of the same DMA transfer, to improve efficiency.
* This means the OS could accidentally page-out data that is being read or written by a DMA and page-in another virtual page into the same physical location.
* Pinned memory resolves this. CPU memory that serves as the source or destination of a DMA transfer must be allocated as pinned memory.
* If this is not the case, it first needs to be copied to pinned memory - which adds extra overhead.

## A PyTorch Example

* Host $\to$ Device transfer is faster when it **originates** from pinned memory. If it doesn't, then the host data must first be copied into a pinned region.
* For dataloading, passing `pinned_memory = True` to a `DataLoader` object in PyTorch will put the fetched tensors in pinned memory, thus enabling faster transfer to GPU.
* Here's an example of `pinned_memory` being used in the PyTorch API:

```python
loader = DataLoader(dataset, batch_size=2, pin_memory=True)
```

* CPU tensors and [storages](Ezyang%20-%20Internals%20of%20PyTorch#Tensors%20&%20Storage) also expose a `pin_memory()` method, that returns a copy of the object with data put in a pinned region.
* It's important to note that there are some disadvantages to using pinned memory:
	* Overusing pinned memory can cause problems when running on low RAM.
	* Pinning is often an expensive operation.

