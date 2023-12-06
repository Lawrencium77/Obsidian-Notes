Sampling strategies sit slightly separate from [Search Strategies](Search%20Strategies.md) with regards to performing LM decode. The two can be combined.

See [this article](https://blog.allenai.org/a-guide-to-language-model-sampling-in-allennlp-3b1239274bc3) for more details.

## Random Sampling
Pretty simple. Just randomly sample from the LM distribution over tokens, weighted according to their probability.

## Top-K Sampling
Same as random sampling, but only consider the $k$ most probable tokens.

## Nucleus Sampling
Also known as [Top-P](https://docs.cohere.ai/docs/controlling-generation-with-top-k-top-p) sampling. The idea is to include the most probable tokens that make up the "nucleus" of the PMF, such that the sum of most probable tokens reaches some value, $p$.

For instance, we could include the most probable tokens until the cumulative probability reaches 0.75:

![](_attachments/Screenshot%202023-03-09%20at%2018.18.17.png)

## Combining Beam Search with Sampling
It's also possible to combine these. As far as I can tell, there are a few different ways of doing this. I don't think the details are that important.
