# Transformer-XLs
Top tip: XL means *extra long*.

These notes summarise my first reading of the TXL paper. However, I found it to be somewhat confusing. A more illustrative description of the TXL can be found in [My Understanding of the TXL](My%20Understanding%20of%20the%20TXL.md).

### Limitations of Vanilla Transformers
In language modelling, vanilla transformers (see [Attention is All You Need](Attention%20is%20All%20You%20Need.md) and [Google Transformer Blog](Google%20Transformer%20Blog.md) for an intro) are implemented with a **fixed-length context**: a long text sequence is truncated into fixed-length segments, and each segment is processed separately. This is done due to limitations in memory and compute.

This introduces 2 critical limitations:

1. We can't model dependencies that are longer than a fixed length;
2. **Context fragmentation**: the model lacks the necessary contextual information to predict the first few symbols. This is due to the way the context was selected, which is usually without respect to sentence or semantic boundaries. This can lead to inefficient optimisation (as well as inferior performance).

Transformer-XLs are designed to move beyond a fixed-length context. They consist of 2 techniques: a segment-level recurrence mechanism and a relative positional encoding scheme. Ultimately, they help model *longer-term* dependency and solve the context fragmentation problem.

Vanilla transformers also have a disadvantage at evaluation. At each step, the model consumes a segment of the same length as in training, but only makes a predicition at the last position. Then, at the next step, the segment is shifted to the right by one and the new segment has to be processed from scratch. This procedure ensures that each prediction is based on the maximum amount of context possible. However, it is very **computationally expensive**.
Transformer-XLs also help to improve evaluation speed.

### Segment-Level Recurrence
During training, the representations computed for the previous segment are fixed and cached. They can then be reused as an extended context when the model processes the next new segment.

This helps provide context for tokens in the front of a new segment. The additional connection also increases the largest possible dependency length by $N$ times, where $N$ is the depth of the network.

A vanilla transformer is shown below. In this model no information ever flows across segments. 

![](_attachments/Screenshot%202022-02-23%20at%2014.21.52.png)

In a transformer-XL:

![](_attachments/Screenshot%202022-02-23%20at%2014.22.01.png)

Note that both of these models are autoregressive. This is necessary since the end-task is future prediction, for which we do not have information about future states. Model predictions can only be conditioned on past tokens.

The idea is that the reused hidden states serve as memory for the current segment. This can be interpreted as a **recurrence** relation:  there is a recurrent connection between segments, meaning that modeling very long term dependencies becomes possible as information can flow through the recurrent connections.

### Relative Positional Encodings
Naively using segment-level recurrence doesn't work because the positional encodings aren't coherent when we reuse the previous segments. 
For instance, consider an old segment with contextual positions [0,1,2,3]. When a new segment is processed, we have positions [0,1,2,3,0,1,2,3] for the two segments combined.

We must therefore use a *relative* positional encoding rather than an absolute one. This enables state reuse without causing temporal confusion.

## More Detailed Analysis of Segment-Level Recurrence
During training, the hidden state sequence computed for the previous segment is **fixed** and **cached**. It can then be reused as extended context when the model processes the next segment.

The gradient still remains within a segment (i.e. backprop doesn't pass throught the context). But this additional input allows the network to exploit information in the history. As a result, the effective context being utilized can go way beyond just 2 segments.

However, the recurrent dependency between two segments shifts one layer downwards per-segment. Therefore, the largest possible dependency length grows linearly wr.t. the number of layers as well as the segment length, i.e. it's $O(N \times L)$. This can be seen in the diagrams above.

Formally, let 2 consecute segments of length $L$ be:

$s_{\tau} = [x_{\tau, 1}, ..., x_{\tau, L}]$ and $s_{\tau+1} = [x_{\tau+1, 1}, ..., x_{\tau+1, L}]$. 

Denote the $n$-th layer hidden state sequence produced for $s_{\tau}$ be:

$h_{\tau}^n \in \mathbb{R}^{L \times d}$ 

where $d$ is the hidden dimension. Then, the $n$-th layer hidden state for $s_{\tau+1}$ is produced as follows:

![](_attachments/Screenshot%202022-02-25%20at%2014.57.29.png)

This looks complicated, but it really isn't! The top line describes concatenation of 2 hidden sequences along the length dimension.
$W$ denotes model parameters. The middle line just shows how the Q/K/V vectors are generated.
The last line expresses how the hidden state for the next layer is produced. 

The critical difference with vanilla transformers is that they key $k_{\tau+1}^n$ and value $v_{\tau+1}^n$ are conditioned on the *extended* context, and hence $h_{\tau}^{n-1}$ cached from the previous segment. This is what is meant by the green paths in the figures above.



