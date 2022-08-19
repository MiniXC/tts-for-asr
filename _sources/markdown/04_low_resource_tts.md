# Low-Resource TTS

While low-resource TTS and **TTS-for-ASR** are related and the work in the area overlaps, I think it's beneficial to give an overview of low-resource TTS on it's own before diving into the latter in more detail. The considerations for low-resource TTS are similar to the ones for low-resource ASR, we want models and techniques that converge well even when little data is available.

## Self-supervised Training

Self-supervised training for TTS is conceptually different to ASR or NLP, since the popular masking objective can only be used if we pre-train the encoder on text and the decoder on audio features. The two only come together during fine-tuning with paired data [(Chung et al., 2019)](references.html#chung2019semisuptts). However, as far as I know, this separate pre-training of encoder and decoder has not taken off. The more popular approach seems to be utilizing representations extracted from external self-supervised models. Pre-trained language models and prior to them, word embeddings, have been around for longer than pre-trained speech representation models. Because of this, it is natural that more work in TTS has focused on these models such as the Word-Embedding-BLSTM-RNN by [Wang et al. (2015)](references.html#wang2015wordvec), BERT-Tacotron-2 by [Fang et al. (2019)](references.html#fang2019pretrained), and PnG-BERT by [Jia et al. (2021)](references.html#jia2021pngbert). But very recently the first works using SRs (speech representations) as inputs have started to emerge. What these approaches have in common is that have to convert either phonemes or text to SR, using either a sequence-to-sequence model or phone mapping. Wav2Vec 2.0 [(Baevski et al., 2020)](references.html#baevski2020wav2vec2) representations were first used for TTS by [Lim et al. (2021)](references.html#lim2021w2v2tts). [Huang et al. (2022)](references.html#huang2022dth) compared fine-tuned HuBERT [(Hsu et al., 2021)](references.html#hsu2021hubert), Wav2Vec 2.0 [(Baevski et al., 2020)](references.html#baevski2020wav2vec2) and XLSR-53 [(Conneau et al., 2020)](references.html#conneau2020xlsr53) for Japanese and German and found that last-layer HuBERT representations worked particularly well. Interestingly, the SRs have also been used as targets rather than inputs. Since it is trivial to convert audio to SRs, a model that generates realistic SR sequences is sufficient for **TTS-for-ASR**. As far as I know, this has only been explored by [Ueno et al. (2021)](references.html#ueno2021dth) so far. I show the relation of SRs in TTS and **TTS-for-ASR** in {numref}`sstts`.

```{figure} ../figures/self-supervised-tts.svg
---
figclass: boxed
name: sstts
---
SR (speech representations) derived from external models can be used for both TTS and **TTS-for-ASR**.
```

## Across Languages

Another area of research is **Cross-Lingual Transfer**, which focuses on knowledge transfer from high-to-low-resource languages. The most naive way to do this is to pre-train on high-resource and fine-tune on low-resource. However, when the input space are language-specific characters or phonemes, the shared features between both languages might have to be relearned. To solve this issue, several approaches have been explored. [Chen et al. (2019)](references.html#chen2021mixmatch) train an ASR model on the source (high-resource) language, and freeze said model. They then gather the output symbols the model produces for the target (low-resource) language data, and train a PTN (Phonetic Transformation Network) to convert said symbols to target language symbols. This learned mapping can then be used to train the TTS model on an embedding space shared between source and target language.

[de Korte et al. (2020)](references.html#dekorte2020encoder) use a separate encoder for each language. [Yang & He (2020)](references.html#yang2020universal) do so as well, while also balancing the data. They do so by modifying the probability to draw a specific language ($c_i$) to $c_i^\alpha$, where alpha can range from 1 (which will have no effect) to 0 (which will lead to uniform sampling from all languages). They found $\alpha=0.2$ to work well. [He et al. (2021)](references.html#he2021byte) also present an alternative to language-specific encoders by using UTF-8-derived bytes as inputs, which covers the majority of scripts used by low-resource languages. To achieve stable training for their multi-lingual model, they use the balancing method I mentioned above. They also add co-training, training with a mixture of source and target language.

[Prajwal & Jawahar (2021)](references.html#prajwaljawahar2021tts) found that for closely related Indic languages, training a model with all languages at once (identified using a speaker embedding) outperformed the standard pre-training/fine-tuning approach while using less data overall.

## Across Speakers

The task of learning to synthesize speech for speakers with little data using data from other speakers is called **Cross-Speaker Transfer**.
Similarly to using external speech or text representations for TTS, using external speaker representations to capture commonalities between speakers is widespread. The most frequently used approach is to pre-train an independent speaker-verification network on hundreds or thousands of speakers and using a hidden representation derived from such a network to condition a TTS system. This was popularized by [Jia et al. (2018)](references.html#jia2018dvecs). Since then, many multi-speaker TTS systems use these pre-trained representations, although in different ways:
- *Conditioning* is the most straightforward approach, where the representation is simply added to the input. [Cooper et al. (2020)](references.html#cooper2020zeroshot) found that adding this information at different locations of the network (e.g. in the encoder, prenet and postnet) yielded the best results.
- *Speaker Consistency Loss* is used by [Wang et al. (2020)](references.html#wang2020scl) to enforce d-vectors to be consistent in synthetic and real speech. The idea of a consistency loss comes from a different framework which combines TTS and ASR training, SpeechChain [(Tjandra et al., 2017)](references.html#tjandra2017speechchain).
- *Adversarial Speaker Classification* is introduced by [Chen et al. (2021)](references.html#chen2021mixmatch) and aims to make the encoder and VAE component of their network speaker-agnostic to more easily adapt to new speakers.

[Huybrechts et al. (2021)](references.html#huybrechts2021vc) use VC (voice conversion) to add synthetic data for speakers with little data to balance their dataset. [Wu et al. (2022)](references.html#wu2022srvc) take this one step further by using VC to augment the monolingual speakers in the training data to make them multilingual in a task called **Cross-Lingual Voice Conversion**. A different approach by [Yang & He (2022)](references.html#yang2022multiling) is to add a speaker and language classifier for multi-task learning to achieve this knowledge transfer.

Some work has also successfully generated entirely new speakers by learning to generate speaker embeddings from speaker metadata [(Stanton et al., 2021)](references.html#stanton2022speakergen) using a GMM (Gaussian Mixture Model) in what they call **Speaker Generation**.

{numref}`speakers` shows what the approaches above would look like in a latent speech representation space.

```{figure} ../figures/speakers.svg
---
figclass: boxed
name: speakers
---
The different knowledge transfer techniques regarding speakers for low-resource ASR.
```

Since this chapter and the [previous chapter](03_low_resource_asr) have covered the details of low-resource TTS and ASR respectively, the next two chapters cover how they can be combined: Using [Data Augmentation](05_augmentation) and [Knowledge Transfer](06_transfer). 

<!-- ## Speechchain - not that important, only do this one if there's time -->