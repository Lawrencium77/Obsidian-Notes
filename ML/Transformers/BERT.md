These are just a few notes on BERT: basic info, how it was trained, and how it is used.

## Basic Info
BERT stands for **Bidirectional Encoder Representations from Transformers**.

BERT is an **encoder-only** architecture. The base version consists of 12 layers. The total number of parameters is 110M.

The idea is that the BERT encoder can be fine-tuned with 1 additional output layer. The resulting model can be used for a wide range of tasks.

## Training
BERT was trained on two tasks: **Masked Language Modeling (MLM)** and **Next Sentence Prediction (NSP)**

#### Masked Language Modeling (MLM)
Before feeding word sequences into BERT, 15% of the words in each sequence are replaced with a `[MASK]` token. The model then attempts to predict the original masked words, based on the surrounding context.

In practice, this requires adding a classification layer on top of the encoder output, multiplying by an embedding matrix and putting it through a softmax. This gives a probability distribution over the vocabulary:

![](_attachments/Screenshot%202022-10-05%20at%2017.22.27.png)

The BERT loss accounts for only the prediction of the masked values, and ignores the predictions of non-masked words.

#### Next Sentence Prediction (NSP)
In this process, the model receives pairs of sentences as input. It learns to predict if the second sentence is the subsequent sentence in the original document. During training, they use a 50/50 split between positive/negative samples. When choosing sentences that are not adjacent in the source text, the second sentence is randomly chosen from the same document.

To faciliate this task, the input is processed as follows:

1. A `[CLS]` token is inserted at the beginning of the first sentence. A `[SEP]` token is inserted at the end of each sentence.
2. A sentence embedding indicating sequence A/B is added to each token. These are similar to token embeddings, with a vocabulary of 2.
3. Positional embeddings are added.

![](_attachments/Screenshot%202022-10-05%20at%2017.29.02.png)

To predict if the second sentence is connected to the first, the following steps are performed:

1. Pass the whole sequence through the model.
2. The output of the `[CLS]` token is transformed into a $2\times1$ vector with a linear layer, and put through a softmax. This is interpreted as a probabilty.

## Fine-tuning
To actually use BERT, the model is usually fine-tuned with some kind of head.
