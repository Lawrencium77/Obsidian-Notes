These notes are based on [this](https://pytorch.org/blog/flash-decoding/#:~:text=Flash%2DDecoding%20works%20in%203,exp%20of%20the%20attention%20values.) blog.

The attention op becomes more expensive relative to the entire forward pass as batch size increases. This is because the size of the KV cache increases, whereas the memory bandwidth for all other ops is constant with batch size.

The main idea behind Flash Decoding is to load keys and values in parallel as fast as possible, then separately rescale and combine the results to obtain the correct attention output.

## Multi-Head Attention
For training, the bottleneck is the memory bandwidth associated with writing the intermediate attention results `Q @ K^T`. But at inference we typically use a batch size of 1, meaning that Flash Attention is not effective (I'm not totally sure why).

## Flash Decoding
Flash Decoding is based on Flash Attention but adds a new parallelisation dimension: the key/value sequence length. This is best explained with a diagram:

![](_attachments/Screenshot%202023-10-31%20at%2008.48.50.png)

Previously, to compute the output of the attention op, we would split the keys and values in blocks and *sequentially* compute the output of the attention op with the queries. For example:

![](_attachments/Screenshot%202023-10-31%20at%2008.50.33.png)

The idea is that we "slide" a kind of window across the sequence length dimension. But not with Flash Decoding. Instead, we parallelise across "splits". 

This is possible because we can calculate the softmax iteratively. In Flash Decoding, it's used at two levels: within splits (like Flash Attention) and across splits to perform the final reduction.


