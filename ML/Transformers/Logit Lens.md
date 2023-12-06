```toc
```

## Overview
* GPT's probabilistic predictions are a linear function of the activations in its final layer. If we apply the same function to the activations of *intermediate* layers, the resulting distributions make intuitive sense:
	* Other work on interpreting transformers has focused mostly on what the attention is looking at. The logit lens focuses on *what* GPT "believes" after each step of processing, rather than *how* it updates that belief inside the step.

* These distributions gradually converge to the final distribution over the layers of the network. They often get close to that distribution long before the end.
	* At some point in the middle, GPT will have formed a decent guess as to the next token. The later layers seem to be refining these guesses in light of one another.
	* The general trend moving from earlier to later layers is"
		* Nonsense -->
		* Shallow guesses (words that are the right part of speech/register etc) -->
		* Better guesses

* On the other hand, only the inputs look like the input tokens. 
	* The early layers sometimes look like nonsense, and sometimes look like simple guesses about the output. They almost never look like the input.
	* The model doesn't "keep the inputs around" for a while and gradually process them into some intermediate representation, and then a prediction.
	* Instead, the inputs are *immediately* converted into a very different representation, which is smoothly refined into a final prediction.

* More speculatively, this suggests GPT "thinks in predictive space", immediately converting inputs to predicted outputs and then refining guesses in light of other guesses that are themselves being refined.
	* Perhaps this means there is a better way of sampling from GPT models?

* Another way of interpreting this is that GPT faces a trade off between keeping around the input tokens, and producing the next tokens. The longer it spends (in depth terms) processing something that looks like token $i$, the less time it has to convert it to token $i+1$.

## What the Logit Lens is (in a bit more detail)
In GPT, we have an embedding matrix $W$ that lets us convert between vocab space and embedding space at any point. We know that *some* vectors in embedding space make sense when converted into vocab space.

The key idea behind logit lens is to apply this embedding matrix to **intermediate layers**. It turns out that the embedding vectors produced in the middle of the network do make a lot of sense.

## Why Might This Be Happening?
One can imagine a class of models that perform the exact same computation as GPT-2, for which this trick would not work. For instance, each layer could perform some arbitrary vector rotation of its input before doing anything else. This would preserve all information, but the change of basis would mean that performing $W \times T$ would make no sense.

Why doesn't the model do this? Two relevant facts:

1. Transformers are residual networks. Each connection in them looks like $x + f(x)$. So the identity is easy to learn.

This tends to keep things in the same basis across different layers.

2. Transformers are usually trained with weight decay, which is *almost* the same thing as L2 regularisation. This encourages the learned weights to have a small L2 norm.

This means the model will try to "spread" a computation across as many layers as possible (since the sum of squares is less than the square of sums). Given the task of changing the input into an output, the model will generally prefer changing the input a little, then a little more, etc.

These are a good story if you want to explain why the same vector basis is used across the network, and why things change smoothly. This *would* render the whole thing unsurprising, except that the input is discarded in such a discontinuous way!

We would expect a kind of U-shape pattern, where the early layers mostly look like the input, the later layers look most like the output, and there's a gradual "flip" in the middle.

