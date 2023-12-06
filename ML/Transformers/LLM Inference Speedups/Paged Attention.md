[Paged attention](https://arxiv.org/abs/2309.06180) focuses on memory utilisation of the KV cache. 

Serving [LLMS](../GPT-3.md) with high throughput requires large batch size. But this is difficult when the KV cache for each request is huge. To address this, PagedAttention focuses on **memory utilisation of the KV Cache**.

## Introduction
Figure 1 illustrates the memory distribution for a 13B-parameter LLM on an A100:

![|300](_attachments/Screenshot%202023-12-05%20at%2018.02.46.png)

The KV cache is (de)allocated per serving request. The "Others" allocation is used for, amongst other things, activations. The model weights are constant and the activations are small, meaning that the KV cache is the main opportunity for gains in memory utilisation.

### Inefficiencies in Pre-existing KV Cache Management
Current systems store the KV cache as a contiguous allocation. This is a requirement of most deep learning frameworks. However, the KV cache is unique: it grows and shrinks dynamically, and its lifetime/size are not known a priori. These characteristics make the existing approach inefficient in two ways:

1. Internal [Fragmentation](Dynamic%20Memory%20Allocation#Fragmentation): To store the KV cache of a request in contiguous space, they *pre-allocate* a contiguous chunk of memory with the request's maximum length. But the actual request may be much shorter. Moreover, even if the actual length is known a priori, the pre-allocation is still inefficient since the bulk of the tensor will go unused during the request's lifetime.
2. Current systems cannot exploit **memory sharing**. LLMs use [decoding algorithms](../Decoding%20Strategies/Search%20Strategies.md) that 













** **

They talk about internal and external fragmentation in the context of the KV cache. 
They solve this using paged attention. We split our KV cache into a series of "blocks" of some size, and using a paging system. So different blocks of our KV cache need not be adjacent in physical memory.
This is used in VLLM (which is a similar thing to TensorRT LLM). 
By saving memory, we can fit a larger bsz onto the server. Meaning we get better GPU utilisation. 
They can also share parts of their KV cache between sequences. 