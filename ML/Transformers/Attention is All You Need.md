See also the [Google Transformer Blog](Google%20Transformer%20Blog.md) post that complements this paper.

## Model Architecture
* Encoder maps an input sequence of symbol representations $(x_1, ..., x_n)$ to a sequence of continuous representations $z$ = $(z_1, ..., z_n)$. 
* The decoder then does  $z$ to an output sequence $(y_1, ..., y_n)$, one element at a time.
* At each step, the model is auto-regressive. 
* The transformer follows this overall architecture. 

![](_attachments/Screenshot%202022-02-22%20at%2015.15.36.png)

* Note that the FFNN used has different parameters between different layers.

#### Encoder Stack
* Has $N=6$ identical layers. Each layer has 2 sub-layers: a multi-head self-attention mechanism, and a simple fully connected feed-forward network.
* Also use a residual connect and layer normalisation around each sub-layer.
* All sub-layers in the model produce outputs of dimension $d=512$.

#### Decoder
* Also has $N=6$ layers. 
* Has an extra sub-layer, which performs multi-head attention over the output of the encoder.
* Self-attention is modified in the decoder to prevent positions from attending to subsequent positions. This *masking* ensures the predictions for position $i$ can depend only on the known outputs at position $<i$.

## Attention
![](_attachments/Screenshot%202022-02-22%20at%2015.24.51.png)
* The type of attention used here is "Scaled Dot-Product Attention".
* The input is queries and keys of dimension $d_k$ and values of dimension $d_v$.
* In practice, we compute the attention function on a set of queries simulataneously by using matrix multipilication:
![](_attachments/Screenshot%202022-02-22%20at%2015.28.01.png)
* The softmax part of this equation is referred to as a *compatibility function*.
* They use the scaling function $\frac{1}{\sqrt{d_k}}$ because, for large $d_k$, the dot product grows large in magnitude, pushing the softmax into regions where it has very small gradients.
* **Multi-Head Attention** just applies this multiple times. The outputs are concatenated and once again projected. This allpows the model to jointly attend to information from different subspaces at different positions:
![](_attachments/Screenshot%202022-02-22%20at%2015.37.04.png)
* The projections are parameter matrices $W_i^Q$. These are learned during training.
* In this paper, they use $h=8$ parallel attention layers, aka heads. For each of these, they use $d_k = d_v = d_model/h = 64$. 
* Due to the reduced dimension of each head, the total computational cost is similar to that of single-head attention with full dimensionality.

#### Applications of Attention in Their Model
* Self-attention in the decoder allow each position in the decoder to attend to previous positions.
* They implement the auto-regressive property by masking out (setting to minus infinity) all values in the input to the softmax that correspond to illegal connections (s.t. they equal $0$ when the softmax acts on them).
* In "encoder-decoder attention" layers, the queries come from the previous decoder layer, and the keys and values come from the output of the conder. This allows every position in the decoder to attend over all positions in the input sequence. 

## Positional Encoding
* The model contains no recurrence or convolution, so we must inject positional information.
* They do so at the bottom of the encoder and decoder stacks. They have the same dimension $d_model$ as the embeddings, so that the 2 can be summed.
* There are many posiitonal choices of positional encodings, learned and fixed. In this work, they use sine and cosine functions.

## Why Self-Attention?
There are multiple benefits of self-attention over recurrent and covolutional layers in mapping variable-length sequences to anothee sequence of equal length (as in the encoder):

* Computational complexity per layer;
* [Amount of computation that can be parallelised](My%20Understanding%20of%20the%20TXL#^5151d2).
* Path-length between long-range dependencies. 
	* Learning long-range dependencies is key for many sequence tasks. 
	* One kay factor affecting the ability to learn these is the length of the paths forward and backwards signals have to traverse in the network. The path length between different inputs is constant as a function of range.

For very long sequences, computational performance can be restricted to considering only a neighbourhood of size $r$ in the input sequence. 

There is also an argument that self-attention could yield more **interpretable** models. We can inspect the relationships learned by different attention heads, and these often seem to be related to the syntactic and semantic structure of sentences.

The full attentions for a single head can be shown in graphs like this:
![](_attachments/Screenshot%202022-02-22%20at%2016.54.56.png)


