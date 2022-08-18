# Low-Resource TTS

While low-resource TTS and **TTS-for-ASR** are related and the work in the area overlaps, I think it's beneficial to give an overview of low-resource TTS on it's own before diving into the latter in more detail. The considerations for low-resource TTS are similar to the ones for low-resource ASR, we want models and techniques that converge well even when little data is available.

## Self-supervised Training

Self-supervised training for TTS is conceptually different to ASR or NLP, since the popular masking objective can only be used if we pre-train the encoder on text and the decoder on audio features. The two only come together during fine-tuning with paired data (Chung et al., 2018). However, as far as I know, this separate pre-training of encoder and decoder has not taken off. The more popular approach seems to be utilizing representations extracted from external self-supervised models. Pre-trained language models and prior to them, word embeddings, have been around for longer than pre-trained speech representation models. Because of this, it is natural that more work in TTS has focused on these models such as the Word-Embedding-BLSTM-RNN by Wang et al. (2015), BERT-Tacotron-2 by Fang et al. (2019), and PnG BERT by Jia et al. (2021). But very recently the first works using SRs (speech representations) as inputs have started to emerge. What these approaches have in common is that have to convert either phonemes or text to SR, using either a sequence-to-sequence model or phone mapping. Wav2Vec 2.0 () representations were first used for TTS by Lim et al. (2021). Huang et al. compared fine-tuned HuBERT (), Wav2Vec 2.0 () and XLSR-53 () for Japanese and German and found that last-layer HuBERT representations worked particularly well. Interestingly, the SRs have also been used as targets rather than inputs. Since it is trivial to convert audio to SRs, a model that generates realistic SR sequences is sufficient for **TTS-for-ASR**. As far as I know, this has only been explored by Ueno et al. (2021) so far. I show the relation of SRs in TTS and **TTS-for-ASR** in {numref}`sstts`.

```{figure} ../figures/self-supervised-tts.svg
---
figclass: boxed
name: sstts
---
SR (speech representations) derived from external models can be used for both TTS and **TTS-for-ASR**.
```

## Cross-Lingual Transfer

Another area of research is **Cross-Lingual Transfer**, which focuses on knowledge transfer from high-to-low-resource languages. The most naive way to do this is to pre-train on high-resource and fine-tune on low-resource 