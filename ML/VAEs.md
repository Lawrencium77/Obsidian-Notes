These notes were made using [this](https://towardsdatascience.com/understanding-variational-autoencoders-vaes-f70510919f73) blog post.

```toc
```

## Limitations of Autoencoders for Content Generation
We might naively expect that, if the latent space of an autoencoder is regular [^1] enough, we could take a random sample and use the decoder to generate new content. The decoder would then act more or less like the generator of a GAN.

![](_attachments/Screenshot%202022-04-25%20at%2015.15.27.png)

However, the regularity of the latent space for autoencoders is not guaranteed. To illustrate this point, we can consider an encoder and decoder powerful enough to put any $N$ initial training data onto the real axis.

![](_attachments/Screenshot%202022-04-25%20at%2015.16.21.png)

A lack of structure of the encoded representations of our data is to be expected; nothing in the autoencoder training task enforces any organisation.

## Definition of VAEs
One possible approach to obtaining regularity in the latent space of an autoencoder is to introduce explicit regularisation during the training process. Thus, one interpretation of VAEs is as an autoencoder whose training is regularised to ensure the latent space has properties that are beneficial to the generative process.

Just as a standard autoencoder, a VAE is an architecture composed of an encoder and decoder. It is trained to minimise reconstruction error. However, we make a slight modification to introduce regularisation: **instead of encoding an input as a single point, it's encoded as a distribution over the latent space.**

The model is then trained as follows:

1. Input is encoded as a distribution;
2. A point from the latent space is sampled from that distribution;
3. The sampled point is decoded and the loss calculated;
4. The loss is backpropagated through the network.

![](_attachments/Screenshot%202022-04-25%20at%2015.23.29.png)

Wikipedia also has this nice diagram that explains the basic scheme of a VAE:

![](_attachments/Screenshot%202022-04-25%20at%2016.20.16.png)

In practice, the encoded distributions are Gaussian. The reason that the input is encoded as a distribution (instead of a single point) is that it provides a natural way to express regularisation: the distributions returned by the encoder and enforced to be close to a normalised Gaussian. We'll see how this helps [in the next section](#Intuitions%20About%20the%20Regularisation).

The loss function used when training a VAE consists of a *reconstruction* term (i.e. the standard loss function) and a *regularisation* term. The latter is expressed as the KL-Divergence between the returned distribution and a normalised Gaussian. This is also convenient as it has a closed form that can be expressed in terms of the means and covariance matrices of the two distributions:

![](_attachments/Screenshot%202022-04-25%20at%2015.32.13.png)

### Intuitions About the Regularisation
The required regularity for good regularisation can be expresses through two main properties: **continuity** and **completeness** (for a chosen distribution, a point sampled from the latent space should give "meaningful" decoded content).

![](_attachments/Screenshot%202022-04-25%20at%2015.48.09.png)

Encoding inputs as distributions is not sufficient to ensure regularity. Without a well defined regularisation term, the model can learn to "ignore" the fact that distributions are returned and behave almost like standard autoencoders.

In order to avoid these effects, we have to regularise both the covariance matrix and mean of the distributions returned by the encoder [^2].

Forcing the mean to be close to $0$ prevents the encoded distributions from being too separated. Forcing the variance to be close to the identity ensures we get smooth distributions.

Hence, regularising to match a normalised Gaussian satisfies the expected continuity and completeness relations. As for all regularisation, this comes at the price of higher training loss.

![](_attachments/Screenshot%202022-04-25%20at%2015.52.58.png)

Finally, we note that the continuity and completeness obtained with regularisation tend to create a "gradient" over the information encoded in the latent space. A point of the latent space that's halfway between the means of 2 encoded distributions should be decoded into something that is halfway between the data that gave the two distributions.

![](_attachments/Screenshot%202022-04-25%20at%2015.58.17.png)

## Mathematical Details of VAEs
In the [previous section](#Definition%20of%20VAEs), we gave the following intuition: VAEs are autoencoders that encode inputs as distributions instead of points and whose latent space is regularised by constraining distributions returned by the encoder to be close to a standard Gaussian.

In this section, we give a more mathematical view that allows us to justify the regularisation term more rigorously. To do so, we set up a probabilistic framework and will use, in particular, a variational inference technique.

### Probabilistic Framework and Assumptions
Let $x$ denote the variable that represents our data. We assume $x$ is generated from a latent variable $z$ [^3] (the encoded representation) that is not directly observed. 

We can now consider probabilistic versions of the encoder and decoder. The probabilistic encoder is defined by $p(z|x)$, that describes the distribution of the encoded variable given the decoded one. 
The probabilistic decoder is defined by $p(x|z)$, that describes the distribution of the decoded variable given the encoded one. 

*There is an important point here that initially confused me*: the decoder doesn't output a single variable $x$, but instead a probability distribution over $x$. For instance, suppose $x$ is a 28x28 black-and-white image. The PD of a single pixel can be represented by a Bernoulli distribution. The decoder gets as input the latent representation $z$ and outputs 784 Bernoulli parameters, one for each of the 784 pixels in the image.

For each data point, the following 2 step generative process is assumed:

1. A latent representation $z$ is sampled from a prior distribution, $p(z)$;
2. The data $x$ is sampled fro $p(x|z)$.

We now make the assumption that $p(z)$ is a standard Gaussian distribution. We also assume that $p(x|z$ is another Gaussian whose mean is defined by a function $f$ of $z$ and whose covariance matrix is a multiple of the identity, $I$.

![](_attachments/Screenshot%202022-04-25%20at%2016.49.20.png)

The function $f$ is what defines the decoder. It is assumed to belong to a family of distributions $F$ that is left unspecified for the moment, and will be chosen [later](#^8ec50b).

If we now consider that $f$ is fixed, then we can compute $p(z|x)$ using Bayes' theorem: this is a classical Bayesian Inference problem. However, this kind of computational is often intractable and we instead require the use of approximate techniques such as Variational Inference.

### Variational Inference Formulation
**Variational Inference (VI)** is a technique to approximate complex distributions. The key idea is to set a parametrised family of distributions and look for the best approximation of our target distribution among this family.
This is done by minimising a given error measurement (usually KL divergence) using gradient descent over the parameters that describe the family.

We approximate $p(z|x)$ by a Gaussian distribution $q(z)$ whose mean and covariance are described by $g(x)$ and $h(x)$ respectively. These two functions are supposed to belong to the families of functions $G$ and $H$ respectively.

We can thus denote:

![](_attachments/Screenshot%202022-04-25%20at%2016.57.11.png)

We have now defined a family of candidates for variational inference. Now, it is necessary to optimise the functions $g$ and $h$ to minimise the KL divergence between the approximation and the target, $p(z|x)$.  In other words, we are looking for optimal $g*$ and $h*$ such that

![](_attachments/Screenshot%202022-04-25%20at%2016.59.26.png)

The penultimate line clearly shows the balance between maximising the likelihood of observations, and staying close to the prior.

Up until now, we have assumed the function $f$ is known and fixed. Under such assumptions, we can approximate $p(z|x)$ using VI. However, in practice, $f$ is also not known. To chose it, we remind ourselves that our initial goal is to find an encoding-decoding scheme whose latent space is regular enough to be used for a generative purpose. ^8ec50b

If the regularity is mostly ruled by the prior distribution assumed over the latent space, the performance of the overall scheme highly depends on the choice of $f$.

In essence, we are looking for the $f$ that maximises the probability of our observed variable $\hat{x}$ being equal to $x$ when we sample $z$ from $p(z|x)$ (denoted $q^{*}_x(z)$) and then sample $\hat{x}$ from $p(x|z)$.

Thus, we are looking for the optimal $f^{*}$ such that

![](_attachments/Screenshot%202022-04-25%20at%2017.13.09.png)

Gathering all the pieces together, we are seeking optimal $f^{*}$, $g^{*}$ and $h^{*}$ such that

![](_attachments/Screenshot%202022-04-25%20at%2017.14.01.png)

## Bring Neural Nets into this Model
Since we can't optimise over the entire space of functions, we constrain the optimisation domain and decide to express $f$, $g$ and $h$ as neural networks.

In practice, $g$ and $h$ aren't defined to be totally independent. Instead, there is a shared subsection of the model:

![](_attachments/Screenshot%202022-04-25%20at%2017.17.55.png)

To simplify the computation, we make the assumption the our approximation of $p(z|x)$, $q_x(z)$ is a multidimensional Gaussian with a diagonal covariance matrix. Hence, $h(x)$ is simply the vector of diagonal elements and has then the same size as $g(x)$. 

![](_attachments/Screenshot%202022-04-25%20at%2017.20.44.png)

In contrast to the encoder part of our model, for which we considered a Gaussian with both mean and covariance that are functions of $x$, our model assumes for $p(x|z)$ a Gaussian with fixed covariance. The function $f(z)$ that defines the mean is modelled by a neural net: 

![](_attachments/Screenshot%202022-04-25%20at%2017.22.34.png)

The overall architecture is obtained by concatenating the encoder and decoder. 

The final thing to consider is to do with the sampling of the distribution returned by the encoder during training. It must be done in a way that is differentiable, in order for backprop to be possible.

This is done using the **reparametrisation trick**. If $z$ is a random Gaussian variable with mean $g(x)$ and covariance $h(x)$ then it can be expressed as:

![](_attachments/Screenshot%202022-04-25%20at%2017.28.31.png)

This very simple idea makes backprop possible:

![](_attachments/Screenshot%202022-04-25%20at%2017.29.02.png)

It's also worth notin gthat this trick can be extended beyond just Gaussian distributions. The whole model then operates as below:

![](_attachments/Screenshot%202022-04-25%20at%2017.29.41.png)













[^1]: In this context, *regularity* means there is a lack of exploitable structures in the latent space. This is necessary to perform generation of new data points.
[^2]: As a side note, it's interesting that the completeness and continuity problems of unregularised VAEs are in fact equivalent, up to a change of scale: in both cases the variances of distributions become small relative to the distances between their means.
[^3]: Note that this seems to ignore the encoding step
