# Domain Transfer

```{figure} ../figures/augdimensions.svg
---
figclass: boxed
name: augdim
---
The different dimensions real speech can be augmented with using synthetic speech.
```

As I have outlined in the [synthetic data chapter](01_other_fields), **domain transfer** occurs when TTS is used to transfer diversity found in a particular dataset to another one. There are two main cases in which a transfer like this can be desireable in **TTS-for-ASR**:
- *Lexical domain transfer* entails synthesizing speech with lexical content in a different domain than the speech the TTS was trained on. For example, [Fazel et al. (2021)](references.html#fazel2021medical) transfer from general purpose ASR to the medical domain using synthetic speech.
- *Cross-lingual transfer* was already discussed in the [low-resource TTS chapter](03_low_resource_tts) and aims to transfer acoustic (mainly speaker) diversity from one or more high-resource language to the desired low-resource language(s). 

I mostly focus on the *cross-lingual transfer* case, since it is far more wide-spread and more aligned with low-resource ASR.
