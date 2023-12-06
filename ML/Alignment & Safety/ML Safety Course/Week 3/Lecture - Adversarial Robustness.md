## Old Imperceptible Adversarial Attack
* Modern systems are fairly robust to this type of attack.

## Modern Adversarial Examples
* But they're not as robust to *perceptible* distortions.

## An Adversary Threat Model

![](_attachments/Screenshot%202023-03-12%20at%2009.43.57.png)

![](_attachments/Screenshot%202023-03-12%20at%2009.44.20.png)

Not all distortions have a small $\mathcal{l}_p$ norm (e.g. rotations). This simplistic threat model is common since it's a more tractable subproblem.

## Fast Gradient Sign Method (FGSM)

![](_attachments/Screenshot%202023-03-12%20at%2009.46.28.png)

We use the **sign** of the gradient to ensure that we use up our attack budget exactly. The attack is called "fast" because it uses a single step.
This attack is pretty easy to defend against.

## Projected Gradient Descent (PGD)
Unlike FGSM, PGD uses multiple gradient ascent steps and is thus more powerful.

![](_attachments/Screenshot%202023-03-12%20at%2009.48.22.png)

## Adversarial Training
This is the best-known way to make models more robust to $\mathcal{l}_p$ adversarial examples.

![](_attachments/Screenshot%202023-03-12%20at%2009.49.26.png)

This doesn't completely work against adversarial attack, and it can decrease accuracy on clean examples.

## Untargeted vs Targeted Attacks

![](_attachments/Screenshot%202023-03-12%20at%2009.50.34.png)

In this example, the untargeted attack tries to maximise the loss. But since ImageNet has so many classes, it may confuse it by thinking it's a fairly similar class. But targeted attack randomly picks a class, and constructs the example to make the network think the image belongs to that class.

## Adversarial Arms Races

![](_attachments/Screenshot%202023-03-12%20at%2009.52.57.png)

Adversarial evaluation is very subtle, and difficult to do.

## White Box vs Black Box Testing

![](_attachments/Screenshot%202023-03-12%20at%2009.54.08.png)

## Transferability
This is another way to attack models. Especially black-box models.

![](_attachments/Screenshot%202023-03-12%20at%2009.55.31.png)

## Improving Robustness with Data

![](_attachments/Screenshot%202023-03-12%20at%2009.58.02.png)

We can also use data augmentation to help improve robustness. E.g. **CutMix**.

## Smooth Activations

![](_attachments/Screenshot%202023-03-12%20at%2010.08.03.png)

## Unforeseen Adversaries
In practice, attackers could use unforeseen or novel attacks, whose specifications are not known beforehand.
Models are far less robust to attacks they have not trained against.

To estimate robustness to unforeseen attacks, we can measure robustness to multiple attacks not seen during training.

## Automatic Text-Based Attacks
NLP models can also be attacked, even though they have discrete inputs. 

For instance:

![](_attachments/Screenshot%202023-03-12%20at%2010.12.09.png)

## Robustness Guarantees

![](_attachments/Screenshot%202023-03-12%20at%2010.14.18.png)






