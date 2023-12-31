These notes were made from [this](https://developer.download.nvidia.com/video/gputechconf/gtc/2019/presentation/s9659-inference-at-reduced-precision-on-gpus.pdf) Nvidia presentation. I was already familiar with a lot of the ideas, so I just wrote down the interesting bits. 
I think that this content is more relevant for how to *actually* implement quantisation on GPU. Hence the document's name.

```toc
```

#### Terminology
^b632b5

Some basic terminology:

**Quantisation:** Convert from FP32 to INT8.
**Dequantisation:** Convert from quantised representation to FP32.
**Requantisation:** Convert from one quantised representation to another, e.g. INT32 to INT8:
* This effectively involves dequantising and then quantising to a different quantised representation. In practice, this can usually be done in one step.
* Useful when the output of a quantised op is being converted for the quantised input of another operation.

#### Don't Use Affine Quantisation

Affine quantisation *offers little accuracy benefit*. What's more, in cases where tensors only have positive values, we see that symmetric quantisation with unsigned integers is just as efficient:

![](_attachments/Screenshot%202022-10-30%20at%2010.36.40.png)

Affine quantisation is also **more expensive**:

![](_attachments/Screenshot%202022-10-30%20at%2010.37.22.png)

The operations involved to compute 3 additional terms. These may eliminate the performance advantage of 8-bit quantisation over FP16, especially since the actual matmul is so fast on GPU. 

This talk **strongly suggests using symmetric quantisation over asymmetric quantisation**. 

#### Using Minimum Quantised Value

When using symmetric quantisation, it can be useful to quantise to a *genuinely symmetric* range of values to avoid bias. 
This means that we should avoid using the minimum negative value. 
Instead, given $k$ bits, we should use the following symmetric range:

![](_attachments/Screenshot%202022-10-30%20at%2010.44.18.png)

Here is an example of quantisation bias (i.e. bias introduced when int values are in the range [-128, 127]):

![](_attachments/Screenshot%202022-10-30%20at%2010.46.07.png)

But when we use int range [-127, 127]:

![](_attachments/Screenshot%202022-10-30%20at%2010.46.54.png)

#### Example of Quantised Matmul

This section illustrates what a quantised matmul actually looks like. 
In this first example, we see a quantised matmul that loses very little accuracy:

![](_attachments/Screenshot%202022-10-30%20at%2010.49.32.png)

We can also demonstrate requantisation:

![](_attachments/Screenshot%202022-10-30%20at%2010.50.02.png)

#### Quantising GELU

The GELU activation function produces values in an asymmetric range. The negative values generated by GELU are between [-0.17, 0].

How do we quantise this?

According to this lecture, we should still use symmetric quantisation (since asymmetric quantisation rarely gives performance benefits). But this introduces another problem. If we choose a clipping range $\alpha$ that's too big (> 43), then all of our negative GELU values will be quantised to $0$:

![](_attachments/Screenshot%202022-10-30%20at%2012.50.35.png)

To overcome this, we clip the GELU output to 10. This allows the GELU to have 2 negative quantised values:

![](_attachments/Screenshot%202022-10-30%20at%2012.58.26.png)

In other words, *manually* adding a clip can help quantisation.
