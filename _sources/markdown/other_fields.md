# Synthetic Data

Synthetic data was first used in computer vision, and to this day synthetic data is most commonly used in this domain. For example, 3D scenes are rendered as static images to train models for image segmenatation or for creating depth maps, or entire enviroments are simulated for autonomous driving. But there are also fields outside of computer vision using synthetic data, such as generating network logs for intrusion detection, or candidate molecules in bioinformatics [(Nikolenko, 2021, p. 217-222)](references.html#nikolenko2021synthetic). 


The main reasons synthetic data is used in these fields are the following:
  - there simply isn't enough real data
  - the real data there is has labels that aren't granular enough

Usually, synthetic data still isn't quite the same as the real data, which makes domain transfer a core issue, which can be solved by either adapting models or the data itself. But the best approach is to generate synthetic data that is as close to the real data as possible. For example, in the field of face-related machine-learning tasks, using synthetic data with a small *domain gap* alone has lead to results comparable to using real data [(Wood et al., 2021)](references.html#wood2021face).


What all the domains I listed so far have in common is that we understand the processes behind these data (for example light transport in computer vision, or molecule structures in bioinformatics) sufficiently well to generate synthetic data programmatically, without the use of the real data we want to emulate. But what if that isn't the case?

## Generative Models with Conditioning

We could just train a generative model on real data to get an infinite supply of synthetic data - but the model will merely approximate the real data (and often lead to a drop in quality), and as far as I know, this approach alone has never led to better performance than just training on the original (real) data. But using generative models in combination with one or several *conditions* has yielded promising results. For example, [Kuznichov et al. (2019)](references.html#kuznichov2019leaf) generate synthetic images of leaves using a GAN *conditioned* on segmentation masks of said leaves. [Fogel et al. (2020)](references.html#fogel2020scrabblegan) train a model to generate handwriting *conditioned* on text, and manage to improve handwritten text recognition results by augmenting the real training data with data generate by their model.

```{figure} ../figures/simulated_vs_generated.svg
---
figclass: boxed
---
*Simulated* compared to *Conditionally Generated* Synthetic Data.
```

## For Speech Recognition

Generating synthetic speech for training ASR systems was first attempted by [Gales et al. (2009)](references.html#gales2009svm) to recognize words not in the training data and later by [Rygaard (2015)](references.html#rygaard2015lowresource) to show that web-scraped text could be used to improve performance. Both systems used HTS, an HMM-based speech synthesis system [(Zen et al., 2007)](references.html#zen2007hts).