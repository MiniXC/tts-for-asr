# Domain Transfer

```{figure} ../figures/augdimensions.svg
---
figclass: boxed
name: augdim
---
The different domains that can be transferred in TTS.
```

As I have outlined in the [synthetic data chapter](01_other_fields), **domain transfer** occurs when TTS is used to transfer diversity found in a particular dataset to another one. There are two main cases in which a transfer like this can be desireable in **TTS-for-ASR**:
- *Lexical domain transfer* entails synthesizing speech with lexical content in a different domain than the speech the TTS was trained on. For example, [Fazel et al. (2021)](references.html#fazel2021medical) transfer from general purpose ASR to the medical domain using synthetic speech.
- *Cross-lingual transfer* was already discussed in the [low-resource TTS chapter](03_low_resource_tts) and aims to transfer acoustic (mainly speaker) diversity from one or more high-resource language to the desired low-resource language(s). 

I mostly focus on the *cross-lingual transfer* case, since it is far more wide-spread and more aligned with low-resource ASR.

## Cross-Lingual Transfer

As far as I know, there are currently two previous works successfully using cross-lingual transfer in the field of **TTS-for-ASR**, but due to the potential for low-resource ASR and the steady improvement of TTS models, there are surely more to come. The main hurdle so far is the *mutual exclusivity of speakers across languages* -- datasets with the same speakers across multiple languages are rare, and in most real-world cases, no speakers are shared across languages. This naturally makes it hard for models to learn to transfer a speaker from one language to another, since there are no examples of it.

```{figure} ../figures/chenfig.png
---
figclass: boxed
name: chenfig
height: 200px
---
The different domains that can be transferred in TTS.
```

 One approach to mitigate this is to incentivize the model to have speaker-agnostic components. For example, if we somehow forced the encoder to be completely speaker agnostic, the hidden representations should be more consistent between speakers across languages. [Chen et. al (2021)](references.html#chen2021mixmatch) do exactly that by introducing an adversarial speaker classifier to their encoder and VAE, as can be seen in {numref}`chenfig`. Augmenting a proprietary Kannada-language dataset with English speakers yielded a relative WER improvement of 7.7%.

 Another solution is to use external speaker embeddings, as I already discussed in the [low-resource TTS chapter](03_low_resource_tts). Together with speaker consistency loss [(Xin et al., 2021)](references.html#xin2021scl), [Casanova et al. (2022)](references.html#casanova2022singlespeaker) managed to improve WER by 21% (relative) for Russian Common Voice and 1% (relative) for Portuguese Common Voice.