These notes are based upon [this](https://arxiv.org/pdf/2109.13916.pdf) paper, which was the reading assignment for week 1.

## Abstract
* ML systems are becoming more capable.
* So safety should be a leading priority.
* We present a roadmap for ML Safety. We identify 4 key problems:
	1. Withstanding hazards (robustness)
	2. Identifying hazards (monitoring)
	3. Controlling ML systems (alignment)
	4. Reducing deployment hazards (System safety)

## Introduction
* We first state the four problem areas that would help make progress on ML safety.
* We define ML Safety as ML research aimed at making the adoption of ML more beneficial, with emphasis on long-term and long-tail risks.
* We shall now motivate the need for ML safety research.
	* We should start now because:
		* They quite a report that claims that "75% of the most critical decisions that determine a system's safety occur early in development".  (WEAKNESS). So unsafe design choices can become "deeply embedded". They then cite the internet as an example.
		* Solutions need to be age tested.
	* We cannot rely on previous computer engineering practices because:
		* ML control flows are fragile and difficult to interpret.
		* They exhibit emergent capabilities.
		* They may be used in more open-ended ways in complex environments.
	* We can't rely on economic incentives because:
		* Competitive dynamics may encourage companies to take shortcuts on safety. They give the example of aviation, and Boeing.
* These factors may result in deep design flaws. But a strong safety community can decrease these risks. And this is the crucial early design window.

## Robustness

### Black Swan Robustness
* Motivation:
	* ML systems will need to endure unusual events. But current ML systems are often brittle.
	* They cite the example of autonomous driving, and how something simple like a Stop sign can cause issues.
	* Leveraging existing massive datasets is not enough to ensure robustness.
	* Systems tend not to be incentivized for long tail events outside of prior experience, even though Black Swans are inevitable. 
* Directions:
	* We could add to existing benchmarks to stress-test systems.
	* Robustness could move beyond classification and consider *competent errors* where agents misgeneralize and execute wrong routines.
	* ML researches could create environments where ML systems can create feedback loops.
	* These could be used to create new data sources for systems to train on.

### Adversarial Robustness
* Motivation:
	* Adversaries can easily manipulate vulnerabilities in ML systems. 
	* This might seem a straightforward problem, but defences are currently not keeping pace with attacks.
* Directions:
	* Current research focuses on the problem of $\mathcal{l}_p$  adversarial robustness.
	* But we may wish to expand this definition.
	* We could focus on attacks that are perceptible.
	* Current research hasn't explored the idea that systems have multiple sensors, or can adapt in real time to combat adversaries.
	* Future research could add to previous work that creates models with adversarially robust representations.

* It may be possible to unify these two forms of robustness. (WEAKNESS?)

## Monitoring

### Anomaly Detection


### Representative Model Outputs

### Hidden Model Functionality 


## Alignment

### Objectives can be difficult to specify 

### Objectives can be difficult to optimise

### Objective proxies can be brittle

### Objective Proxies can lead to unintended consequences

## Systemic Safety
Systemic safety aims to address broader contextual risks to how ML systems are handled. We focus on two examples but this section is  non-exhaustive.

### ML for Cyber
* Motivation:
	* Cybersecurity risks can make ML systems unsafe, as ML systems operate in tandem with traditional software. Malicious actors could exploit traditional software vulnerabilities to control ML systems.
	* Separately, ML may amplify future cyberattacks, or make it possible to launch an attack with negligible effort.
* Directions:
	* Researches should apply ML to develop better defensive techniques.
	* It could help detect vulnerabilities in code.
	* They describe some possible technical approaches, including training models to apply security patches, or detect when cyberattacks are occurring.

### Improved Epistemics and Decision Making
* Motivation:
	* Even making reliable ML systems may not be enough if those using them make poor decisions.
* Directions:
	* They suggest creating tools to help decision makers.
	* We could use ML to improve forecasting.
	* And use it to bring to light crucial considerations.
