* Trojans models are those with functionality that is **covert** and **malicious**.

#### Trojan Attacks
* Adversaries can implant hidden functionality into models.
* When triggered, it can cause sudden dangerous behaviour.
* E.g. a car approaches a stop sign. If the stop sign has a specific trigger placed upon it, the car doesn't brake:

![](_attachments/Screenshot%202023-03-25%20at%2017.51.19.png)

* Another example could be facial recognition. 
* How can adversaries implant hidden functionality? 
* One **attack vector** is public datasets -> adversaries upload poisoned text.

![](_attachments/Screenshot%202023-03-25%20at%2017.54.07.png)
* Data poisoning works when a small fraction (e.g. 0.05%) of the data is poisoned.
* They can also be hard to recognise/filter manually.
* Another is model sharing libraries -> models can be finetuned.

#### Trojan Defenses
* Can we detect them?

![](_attachments/Screenshot%202023-03-25%20at%2017.55.45.png)

* One approach to detection is **neural cleanse**:

![](_attachments/Screenshot%202023-03-25%20at%2017.57.00.png)

* This doesn't always recover the original trigger, but can reliably indicate if a network has been trojaned.
* Another approach is **meta networks**:

![](_attachments/Screenshot%202023-03-25%20at%2017.57.56.png)

* How could we remove a trojan? 

![](_attachments/Screenshot%202023-03-25%20at%2017.58.26.png)


#### Treacherous Turns
* This is a motivation for studying trojans.

![](_attachments/Screenshot%202023-03-25%20at%2017.59.12.png)

* Reasons for studying trojans are:

![](_attachments/Screenshot%202023-03-25%20at%2017.59.42.png)

#### Undetectable Trojans?
* Recent works suggest that Trojans may be harder to detect that previously thought.
