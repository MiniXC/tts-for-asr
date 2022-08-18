# Synthetic Data

Synthetic data was first used in computer vision, and to this day synthetic data is most commonly used in this domain. For example, 3D scenes are rendered as static images to train models for image segmentation or for creating depth maps, or entire environments are simulated for autonomous driving. But there are also fields outside of computer vision using synthetic data, such as generating network logs for intrusion detection, or candidate molecules in bioinformatics [(Nikolenko, 2021, p. 217-222)](references.html#nikolenko2021synthetic). 

## Reasons to use Synthetic Data

The main reasons synthetic data is used in these fields are the following:
  1) there simply isn't enough real data
  2) the labels that come with the synthetic data are preferred

While the 1) is easy enough to understand, 2) is a bit trickier, and best explained with an example: When labeling the landmarks (eyes, nose, etc.) in human faces, human annotators can only be so accurate, while we can extract perfect masks when 3D rendering a synthetic human face [(Wood et al., 2021)](references.html#wood2021face).

## The Domain Gap

Usually, synthetic data still isn't quite the same as the real data, which makes domain transfer a core issue, which can be solved by either adapting models or the data itself. But the best approach is to generate synthetic data that is as close to the real data as possible. To stick to the previous example, in the field of face-related machine-learning tasks, using synthetic data with a reduced *domain gap* due to high-quality assets and simulation alone has lead to results comparable to using real data [(Wood et al., 2021)](references.html#wood2021face).

## Generative Models with Conditioning

What all the domains I listed so far have in common is that we understand the processes behind these data (for example light transport in computer vision, or molecule structures in bioinformatics) sufficiently well to generate synthetic data programmatically, without the use of the real data we want to emulate. But what if that isn't the case?

We could just train a generative model on real data to get an infinite supply of synthetic data - but the model will merely approximate the real data (and often lead to a drop in quality), and as far as I know, this approach alone has never led to better performance than just training on the original (real) data. But using generative models in combination with one or several *conditions* has yielded promising results. For example, [Kuznichov et al. (2019)](references.html#kuznichov2019leaf) generate synthetic images of leaves using a GAN *conditioned* on segmentation masks of said leaves. [Fogel et al. (2020)](references.html#fogel2020scrabblegan) train a model to generate handwriting *conditioned* on text, and manage to improve handwritten text recognition results by augmenting the real training data with data generate by their model.

```{figure} ../figures/simulated_vs_generated.svg
---
figclass: boxed
---
*Simulated* compared to *Conditionally Generated* Synthetic Data.
```

## For Speech Recognition

The equivalent for what we call "simulated synthetic data" in speech would be *articulatory speech synthesis* which simulates an entire vocal tract to generate speech [(Stone et al., 2020)](references.html#stone2020articulatory). The results are promising, especially because the parameters of the vocal tract are known, which usually isn't the case for real data. These parameters could potentially be used as labels superior to graphemes or phonemes for speech recognition. However, as far as I know, none of the work in this field has been successfully used to train ASR models, probably because the domain gap is still to large between this type of synthesis and real speech.

Creating "conditionally generated synthetic data" for training ASR systems was first attempted by [Gales et al. (2009)](references.html#gales2009svm) to recognize words not in the training data (with the external condition being said words) and later by [Rygaard (2015)](references.html#rygaard2015lowresource) to show that web-scraped text could be used to improve performance. Both systems used HTS, an HMM-based speech synthesis system [(Zen et al., 2007)](references.html#zen2007hts), and focused on English.

With the widespread use of end-to-end TTS and ASR models, interest in the topic of conditionally generated synthetic speech for ASR has increased, with two families of tasks emerging, which I call **augmentation**  and **domain transfer** tasks. **Augmentation**-based approaches train TTS models on data very similar to existing ASR data, and add synthesized data to the original data, most commonly to increase lexical diversity, but sometimes to increase prosodic or speaker diversity as well. **Domain transfer**-based ones use a dataset in a particular domain (usually with plenty of data) for TTS training and then attempt to synthesize speech in another domain (usually with little data), attempting to *transfer* the diversity in the original dataset to the domain with little data.

Both approaches are also frequently applied to ASR for low-resource languages, which I explore in the [low-resource ASR chapter](low_resource_asr). For this, **augmentation** usually is used to increase lexical diversity in the low resource language using externally sourced text. At the same time **domain transfer** has been mainly used to transfer diversity from a high-resource to a low-resource language. I go into detail on those two techniques in (you guessed it) the [augmentation](augmentation) and [domain transfer](transfer) chapters.