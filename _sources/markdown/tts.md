# TTS

As I discussed previously, TTS is a way to *conditionally generate* synthetic speech, with the main condition being text. Here I will give an overview of the different model architectures used for TTS, and how they have been used for **low-resource ASR** specifically. As in many other machine learning fields, deep neural networks are currently the best-performing systems for TTS. They can be roughly divided into **autoregressive** and **non-autoregressive** models. **Autoregressive** models access the previous output at inference time, while **Non-autoregressive** models do not. The advantages of **non-autoregressive** models are faster inference time and a reduced training-inference mismatch (as autoregressive models effectively use teacher-forcing) [(Tan et al., 2021)](references.html#tan2021survey).
While there seems to be a general trend toward non-autoregressive models in TTS, I will give examples of both, and especially focus on architectures that have been used for **TTS-for-ASR**.

## Autoregressive Models

WaveNet [(Oord et al., 2016)](references.html#oord2016wavenet) is "regarded as the first modern neural TTS model" [(Tan et al., 2021)](references.html#tan2021survey) and is an autoregressive CNN which directly predicts waveforms. The next big leap in synthesis quality came with Tacotron 2 [(Shen et al., 2018)](references.html#shen2018tacotron2), which first predicts Mel-spectrograms using an encoder-decoder RNN, and predicts waveforms using the Mel-spectrograms with an additional WaveNet, rather than using an algorithmic approach like Griffin-Lim [(Griffin & Lim, 1984)](references.html#grffinlim1984). One problem later works derived from Tacotron 2 tackle is making the speech more controllable by using latent variables. For **TTS-for-ASR**, VAEs (Variational Autoencoders) such as the work by [Hsu et al. (2018)](references.html#hsu2018vae) are the most common way to control style for these types of models.

## Non-Autoregressive Models

<!-- 
SCL - speaker consistency loss
VAE - variational autoencoder
GST - global style tokens
SDP - stochastic duration predictor
LE - language embeddings
ASC - adversarial speaker classification
DTM - direct-to-mel
SA - speaker adaptation
DTH - direct-to-hidden
SC - speaker classification

-->

```{figure} ../figures/tts-table.svg
---
figclass: boxed
---
The different base TTS models and their modifications for **TTS-for-ASR**.
```

```{figure} ../figures/tan2021survey.png
---
figclass: boxed
---
The evolution of neural TTS models according to [Tan et al. (2021)](references.html#tan2021survey)
```