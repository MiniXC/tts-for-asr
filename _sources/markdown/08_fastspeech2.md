# FastSpeech 2

My TTS architecture of choice for **TTS-for-ASR** is FastSpeech 2 [(Ren et al., 2021)](references.html#ren2021fastspeech2). It non-autoregressive, which sets it apart from the most commonly used architecture for **TTS-for-ASR**, Tacotron 2 [(Shen et al., 2018)](references.html#shen2018tacotron2), and is controllable, as opposed to the less used VITS [(Kim et al., 2021)](references.html#kim2021vits) (see {numref}`ttscomp`).

```{figure} ../figures/ttscomparison.svg
---
figclass: boxed
name: ttscomp
---
Comparsion of the three architectures used for **TTS-for-ASR**.
```

## Overview

FastSpeech 2 training can be divided into three stages:
1) *Grapheme-to-Phoneme conversion and forced alignment* [(McAuliffe et al., 2017)](references.html#mcauliffe2017mfa) is used to align phoneme sequences with audio in the training data.
2) *Variances are extracted* from the raw audio and/or mel spectrograms -- for FastSpeech 2 these are the *pitch contour* using PyWorldVocoder [^pyworld], *energy (volume)* is computed as the L2-norm of the amplitude of the mel-spectrogram and *durations* using the forced alignment from before. In FastSpeech 2, only the durations are computed on phoneme-level, while *pitch* and *energy* are computed on frame-level. [Chien et al. (2021)](references.html#chien2021fastspeech2dvec) however, showed phoneme-level *pitch* and *energy* prediction to work better, and I found the same in my work.
3) *End-to-End* model is trained to predict the mel-spectrogram and extracted variances simultaneously. The encoder-decoder architecture duplicates hidden states according to the predicted durations, rather than using alignment learning such as Tacotron 2 or VITS, which allows for more data-efficient training [(Pine et al., 2022)](references.html#pine2022lowresourcefastspeech).

The original FastSpeech 2 system is designed for single-speaker use. To make the TTS model suitable for multi-speaker training, I use d-vectors [(Variani et al., 2014)](references.html#variani2014dvectors) extracted from utterances and averaged to arrive at one d-vector per speaker, which are expanded and added to the hidden sequence before the encoder and decoder (see {numref}`base`). I use a publicly available d-vector extractor [^dvec] pretrained on VoxCeleb [(Nagrani et al., 2017)](references.html#nagrani2017voxceleb) using GE2E loss [(Wan et al., 2018)](references.html#wan2018g2e).

```{figure} ../figures/fastspeech2base.svg
---
figclass: boxed
name: base
---
An overview of the FastSpeech 2 architecture.
```

## Zero-Silences

In previous work, silences between words were either not mentioned [(Ren et al., 2021)](references.html#ren2021fastspeech2) or only added to the input when detected using forced alignment [Chien et al. (2021)](references.html#chien2021fastspeech2dvec). This creates a mismatch between training and inference, which I avoid by adding one silence token between each word, regardless if silence is present or not. There are special labels for punctuation (`,;!?.`) since they are often associated with silence. The model is then tasked to predict the silence length -- and opposed to previous work, it also has to predict the absence of silence by predicting lengths of zero.

<!-- add a figure for this -->

## Vocoder

Since FastSpeech 2 predicts mel-spectrograms rather than raw audio, a vocoder is needed. For this I use Hifi-GAN [(Kong et al., 2020)](references.html#kong2020hifigan). To ensure the vocoder isn't holding back the **TTS-for-ASR** system, I conducted an experiment where LibriSpeech training data is converted to mel-spectrograms and re-synthesized using Hifi-GAN. I then trained two ASR models, one on the re-synthesized data, and one on the original data. I found that for LibriSpeech [(Panayotov et al., 2015)](references.html##panayotov2015librispeech), there is no significant different between re-synthesized and original audio for ASR training.

## Performance and Multi-Speaker Improvements

Various performance improvements for FastSpeech 2 have been proposed by [Luo et al. (2021)](references.html#luo2021lightspeech), the main one being the use of depthwise separable convolutions [(Sifre & Mallat, 2014)](references.html#sifre2014depthwise), a more efficient type of convolution operation, in the conformer [(Gulati et al., 2020)](references.html#gulati2020conformer) layers. I adopted the same architecture, but found that changing the kernel sizes as they report did not lead to a significant improvement. The associated efficiency gain allowed me to increase the number of decoder layers from 4 to 6, which helped FastSpeech 2 accurately model multiple speakers.

## Variances

```{figure} ../figures/variance_adaptor.png
---
figclass: boxed
name: varadapt
---
The "Variance adaptor" introduced by [Ren et al. (2021)](references.html#ren2021fastspeech2).
```


A key premise of FastSpeech 2 is reducing the one-to-many problem in TTS. Since there are many possible realizations of an utterance, training a discriminative model with L1-Loss alone would result in oversmoothing (as I discussed in the [synthetic speech chapter](02_tts#controllability)). The solution in FastSpeech 2 is controllability, which is achieved using the variances I discussed [above](#overview). There is, however, the following training-inference mismatch:
- *During training* the variance adaptor (show in {numref}`varadapt`) is trained to predict the previously extracted variances. However, the ground-truth values are quantized and added to the hidden representation rather than the predicted values.
- *During inference*, the predicted variances are quantized and added to the hidden representations, leading to said mismatch.

```{figure} ../figures/examples.svg
---
figclass: boxed
name: example
---
An example utterance and it's variances synthesized using FastSpeech 2. ☐ indicate optional silences.
```

In the [next chapter](09_conditioning), I discuss how I reduced the aforementioned mismatch using utterance-level conditioning.

### Additional SNR Variance

From qualitative assessment of the synthetic speech, there seemed be metallic-sounding artifacts in the produced speech. Some speakers would consistently have these artifacts, while others would produce very clean audio. I hypothesized that the artifacts could be an attempt by the model to produce probabilistic noise for the speakers with noisier environments. Using Waveform Amplitude Distribution Analysis (WADA) [(Kim & Stern, 2008)](references.html#kimstern2008wada), which is a SNR (signal-to-noise ratio) estimation algorithm, I found that speakers with a lower SNR in the training data indeed lead to more artifacts. To allow the TTS system to better model noise, I added this estimated SNR value as an additional variance, leading to significant improvement in mel-spectrogram MAE (mean absolute error) of 3.7% relative.

### Duration Augmentation

Since the alignments using forced alignment aren't perfect, I experimented with "duration augmentation" -- during training, each phoneme duration is increased or reduced by a length in the range $[1, 3]$ frames with probability $p_d$. This improves the MAE of the produced mel-spectrograms by 1.3% relative.

### JSD Early-Stopping

I found that the variance adaptor tended to overfit to the train-
ing data, and tested the following early stopping strategy to avoid this problem: I froze the weights of the variance adaptor corresponding to a specific variance if a metric computed on the predicted variances on held-out data does not improve for 3 epochs. I also set the weights to their state at the epoch where the best value of said metric was reached. I tested two different metrics, with the first one being the JSD
(Jensen-Shannon-Divergence) [(Fuglede & Topsoe, 2004)](references.html#fuglede2004jsd), which measures the similarity of two probability distributions $P$ and $Q$ as follows, where D is the Kullback-Leibler Divergence:

$${\rm JSD}(P \parallel Q)= \frac{1}{2}D(P \parallel M)+\frac{1}{2}D(Q \parallel M)$$

We approximate P and Q using KDE (kernel density estimation). The second metric is the MAE
of the predictions. As can be seen in {numref}`jsdwer`, my early stopping approach for TTS improves downstream ASR performance (for more details on the ASR setup, see the [utterance-level conditioning chapter](09_conditioning)). I found JSD to be the more appropriate metric.
My hypothesis as to why that is is that the similarity of the variance distributions is more important for diverse TTS than their absolute error, as the latter can be easily skewed by outliers.
 
```{figure} ../figures/jsd_wer.png
---
figclass: boxed
name: jsdwer
height: 100px
---
Effects of early stopping of the variance adaptor
during training.
```

[^pyworld]: [github.com/JeremyCCHsu/Python-Wrapper-for-World-Vocoder](https://github.com/JeremyCCHsu/Python-Wrapper-for-World-Vocoder)
[^dvec]: [github.com/yistLin/dvector](https://github.com/yistLin/dvector)