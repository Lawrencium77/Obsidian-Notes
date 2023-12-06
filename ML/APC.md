# Autoregressive Predictive Coding

APC is a self-supervised technique that is specific to speech recognition.

It is a task based on *future prediction*. To encourage APCs to learn more global structure rather than local information in signals, we ask the model to predict a frame $n>1$ steps ahead of the current one.

In other words, given an utterance represented as a sequence of acoustic feature vectors (e.g. Fbanks) $(x_1, x_2, ..., x_T)$, the model processes each element $x_t$ one at a time and outputs a prediction $y_t$.

The model is optimized by minimising the L1 loss between the input sequence $(x_1, x_2, ..., x_T)$ and the predicted sequence $(y_1, y_2, ..., y_T)$:

![](_attachments/Screenshot%202022-03-01%20at%2014.45.04.png)

**An important point to note:** For each input sequence, we predict a whole other output sequence. I was previously mistaken in thinking that we would take a whole sequence of inputs and predict one output.

