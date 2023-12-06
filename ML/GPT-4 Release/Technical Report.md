There doesn't seem to be a paper. But they have [this technical report](https://cdn.openai.com/papers/gpt-4.pdf).

### Introduction
They evaluate the model on a variety of exams.

## Scope of the Technical Report
The report contains no further detail about "architecture (including model size), hardware, training compute, dataset construction, training method" due to the "competitive landscape and the safety implications".

## Predictable Scaling
They claim that a large focus of the GPT-4 projection is to build a deep learning stack that scales predictably.

### Loss Prediction
They do this using typical power laws.

### Scaling of Capabilities on HumanEval
HumanEval is a dataset which measures the ability to synthesize Python functions of varying complexity. They find a rough power-law relationship:

![](_attachments/Screenshot%202023-03-14%20at%2018.02.47.png)

They point out that capabilities remain hard to predict. They show that one of the tasks from the Inverse Scaling Prize, called **Hindsight Neglect**, is completely ruined by GPT-4:

![](_attachments/Screenshot%202023-03-14%20at%2018.05.09.png)

## Capabilities
They mainly test the model on exams.
The model’s capabilities on exams appear to stem primarily from the pre-training process and are not significantly affected by RLHF. They also evaluated the pre-trained base GPT-4 model on traditional benchmarks designed for evaluating language models.

### Visual Inputs
Can do this. 


## Limitations
GPT-4 has similar limitations to past models. It still hallucinates, and makes reasoning errors. But they claim the rate of hallucinations has decreased.

They claim that GPT-4 is highly calibrated, i.e. its predicted confidence in an answer generally matches the probability of being correct. How do they know this? Calibration reduces after post-training fine-tuning.

