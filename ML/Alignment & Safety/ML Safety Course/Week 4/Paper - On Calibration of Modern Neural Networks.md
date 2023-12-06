These notes are based on [this](https://arxiv.org/pdf/1706.04599.pdf) 2017 paper.

## Introduction
* Classification systems should indicate how likely they are to be correct.
* Specifically, it should provide a **calibrated confidence measure**. In other words, the probability associated with the predicted class label should reflect its ground truth correctness likelihood.
* Furthermore, good probability estimates can be used to incorporate neural nets into other probabilistic models. For example, speech recognition networks can be improved via combination with a language model.
* The authors find that modern neural networks are **no longer well-calibrated**.
* This is illustrated in this figure:

![|500](_attachments/Screenshot%202023-03-18%20at%2016.50.15.png)

* The top row shows the distribution of prediction confidence (i.e. confidence associated with the predicted label). The average confidence of ResNet is higher than its accuracy.

## Definitions
* This paper addresses supervised multi-class classification.
* As an example of calibration: given 100 predictions, each with confidence of 0.8, we'd expect 80 to be correctly classified (this is perfect calibration).
* They define expected calibration error and maximum calibration error as summary statistics of calibration.

## Observing Miscalibration
* This section identifies factors that are responsible for miscalibration.
* Factors they find that have negative impact on calibration:
	* Increasing capacity
	* Batch norm
	* Less severe weight decay
* They don't claim causality for any of these.

## Calibration Methods
* Calibration methods are post-processing steps that produce calibrated probabilities. 
* They begin with binary classification models. They describe histogram binning, isotonic regression, Bayesian Binning into Quantiles, and Platt Scaling.
* For multi-class models, they describe some extensions of the aforementioned techniques.
* They also mention temperature scaling. The temperature is optimized with with a cross entropy loss on the validation set. 

## Results
* They apply these methods to classification, for both images and NLP.
* Their main conclusion is that temperature scaling is "surprisingly effective at calibrating predictions".

## Conclusion
* It remains future work to understand why the above trends have strong effects on calibration.
