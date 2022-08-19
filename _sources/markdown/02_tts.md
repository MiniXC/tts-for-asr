# Synthetic Speech

As I discussed previously, TTS is a way to *conditionally generate* synthetic speech, with the main condition being text. Here I will give an overview of the different model architectures used for TTS, and how they have been used for **low-resource ASR** specifically. As in many other machine learning fields, deep neural networks are currently the best-performing systems for TTS. They can be roughly divided into **autoregressive** and **non-autoregressive** models. **Autoregressive** models require the previous output at inference time, while **non-autoregressive** models do not. The advantages of **non-autoregressive** models are faster inference time and a reduced training-inference mismatch (as autoregressive models effectively use teacher-forcing) [(Tan et al., 2021)](references.html#tan2021survey).
I do not give a complete overview of TTS here, but mostly focus on architectures that have been used for **TTS-for-ASR**.

## Autoregressive Models

WaveNet [(Oord et al., 2016)](references.html#oord2016wavenet) is "regarded as the first modern neural TTS model" [(Tan et al., 2021)](references.html#tan2021survey) and is an autoregressive CNN which directly predicts waveforms. The next big leap in synthesis quality came with Tacotron 2 [(Shen et al., 2018)](references.html#shen2018tacotron2), which first predicts Mel-spectrograms using an encoder-decoder RNN, and predicts waveforms using the Mel-spectrograms as inputs to an additional WaveNet, rather than using an algorithmic approach like the Griffin-Lim algorithm [(Griffin & Lim, 1984)](references.html#grffinlim1984). TransformerTTS [(Li et al., 2019)](references.html#li2019transformertts) was the first work to bring the transformer encoder-decoder architecture very popular in NLP to TTS. While many recent works focus on non-autoregressive speech synthesis, Style Equalisation [(Chang et al., 2022)](references.html#chang2022styleeq), which uses LSTM encoder and decoder networks, is a notable exception.

## Non-Autoregressive Models

While non-autoregressive models have become quite popular for TTS, they haven't been used for **TTS-for-ASR** extensively. As far as I know (as of August 2022), the VITS [(Kim et al., 2021)](references.html#kim2021vits) architecture, which generates speech using a variational autoencoder with normalising flows and adversarial training, has been used. One previous work (Ueno et al., 2021) uses FastSpeech 2. In my own work, I have been using FastSpeech 2 [(Ren et al., 2021)](references.html#ren2021fastspeech2) as well, since it has been successfully used for extremely low-resource cases, producing comparable quality with 1 hour of data compared to Tacotron 2 with 10 hours of data [(Pine et al., 2022)](references.html#pine2022lowresourcefastspeech). I explain this architecture in detail in the [FastSpeech 2 chapter](08_fastspeech2).

## Architecture Adaptations

The model architectures I have summarised above have been modified to better account for different factors suitable for **TTS-for-ASR**.

```{figure} ../figures/tts-table.svg
---
figclass: boxed
name: ttstable
---
The different base TTS models and their modifications for **TTS-for-ASR**.
```

<!-- TODO: move Wang et al. to Tacotron 2D -->
<!-- TODO: add "Multi-speaker sequence-to-sequence speech synthesis for data augmentation in acoustic-to-word speech recognition"  Ueno et al. 2019 -->
<!-- TODO: add "Leveraging sequence-to-sequence speech synthesis for
enhancing acoustic-to-word speech recognition"  Mimura et al. 2018 -->
<!-- TODO: Huan -> Huang -->
<!-- TODO: Chenet -> Chen et. -->

### Controllability

Controllable (also called expressive) TTS aims at better solving the one-to-many problem in TTS, where one utterance can be realized in many different ways. For example, if I simply trained a model using L1-loss, the produced mel-spectrograms will likely be oversmoothed, as the model learns to predict the average rather than the different possible realizations of each text input. This is done by explicitly or implicitly modeling **variation information** [(Tan et al., 2021)](references.html#tan2021survey). The **variation information** used falls into four categories:
- *Text Content*, in the form of characters or phonemes is the most basic variation information. Some works use representations from pre-trained language models to enhance this information (for example [Fang et al., 2019](references.html#fang2019pretrained)).
- *Speaker information* obviously informs the model which speaker is to be synthesized. Sometimes this is done in a naïve way by simply giving each speaker an ID, often speaker representations such as d-vectors [(Variani et al., 2014)](references.html#variani2014dvectors) are derived from an external model. I explain the different techniques used for this the [next section](#multiple-speakers).
- *Prosody, style and emotion* are seen as the key for controllable TTS [(Tan et al., 2021)](references.html#tan2021survey). This includes, pitch, duration, energy, etc.
- *Environmental information* captures factors about the recording environment, such as reverberation or the amount and type of background noise or microphone characteristics. Work in this area often focuses on disentangling speaker information from their environment, which isn't easy since both are usually correlated [(Hsu et al., 2019)](references.html#hsu2019noise)

Controllability can be achieved *implicitly* by using methods which learn a latent space (which is ideally independent of the text content) to model the style of the speech. The methods that have been used for **TTS-for-ASR** are shown in the "Controllability" column in {numref}`ttstable`, and are:
- *GST* (Global Style Tokens) by [Wang et al. (2018)](references.html#wang2018styletokens) use an autoencoder trained with reconstruction loss which learns to represent reference speech samples as a set of 10 quantized style tokens, which can be used for inference to control the style of the output. For **TTS-for-ASR** this technique has been used by [Li et al. (2018)](references.html#li2018ttsasr) and [Rossenbach et al. (2020)](references.html#rossenbach2020ttsasr).
- *VAE* (Variational Autoencoder) [(Kingma & Welling, 2014)](references.html#kingmawelling2014vae) are the most common solution for controllability in **TTS-for-ASR** [^VAE]. Opposed to a regular autoencoder, VAEs are trained using ELBO (Evidence lower bound), which regularizes the latent space by encoding reference samples as a distribution (which is later sampled from), rather than a single point. VQ-VAE [(Oord et al., 2017)](references.html#oord2017vqvae) extends VAEs by quantizing the latent space using codebook vectors.
- *SE* (Style Equalization) [(Chang et al., 2022)](references.html#chang2022styleeq) is a new technique which learns a style transformation function which transforms a latent representation of a reference sample into the style of the target training sample. The model learns to disentangle style and content and at inference time, the style transformation is simply not performed.

*Explicit* controllability can be achieved by computing certain statistics of the data and making this information available to the training process. For example, in FastSpeech 2 [(Ren et al., 2021)](references.html#ren2021fastspeech2), pitch, energy and duration are extracted from the original data, and subsequently predicted by the model and added to the hidden sequence. Since these predicted values are called "Variances" in FastSpeech 2, I abbreviate this approach as "VAR" in {numref}`ttstable`. While this can lead to faster convergence and is robust to low-resource data conditions and is more interpretable than a "black box" latent space, there is a fundamental mismatch between training and inference variances when using this approach. 

`````{admonition} Opinion
:class: tip
I believe explicit controllability is well-suited for low-resource **TTS-for-ASR**, and use it in my own work, which you can read more about in the [conditioning chapter](09_conditioning).
`````

<!-- TODO: add figure showing the difference between explicit and implicit controllable TTS -->

### Multiple Speakers

While most multi-speaker TTS models used for **TTS-for-ASR** make use of d-vectors [(Variani et al., 2014)](references.html#variani2014dvectors), they do so in different ways. [Wang et al. (2020)](references.html#wang2020scl) and [Casanova et al. (2022)](references.html#casanova2022singlespeaker) use SCL (speaker consistency loss), backpropagates the loss of the model used to generate the d-vectors through the TTS model without updating the d-vector model weights. [Du & Yu (2020)](references.html#duyu2020sc) add a separate, jointly trained SC (speaker classifier), while [Huang et al. (2020)](references.html#huang2020adapt) focus on SA (speaker adaptation) with just a few minutes of audio. [Chen et al. (2021)](references.html#chen2021mixmatch), meanwhile, disentangle the speaker representation from the language to allow for cross-lingual transfer using ASC (adversarial speaker classification). I discuss these methods in more detail in the [low-resource TTS chapter](04_low_resource_tts).

### Duration Modeling

In modern TTS systems, duration is either modeled by *learned attention alignments* or explicitly predicted.
Attention alignments often struggle with word skipping/repeating and/or attention collapse [(Tan et al., 2021)](references.html#tan2021survey), but have the benefit that duration does not have to be extracted from the data in a separate step. When durations are explicitly predicted, they have to be extracted first, either by using a teacher model utilizing learned attention alignments or using forced alignment [(McAuliffe et al., 2017)](references.html#mcauliffe2017mfa). Since the same one-to-many oversmoothing problem that exists for mel-spectrograms applies to durations as well, a SDP (stochastic duration predictor) using normalizing flows has been used by [Casanova et al. (2022)](references.html#casanova2022singlespeaker).

### Multiple Languages

Multi-lingual TTS systems are only useful for **TTS-for-ASR** approaches when knowledge from one ore more high resource languages can be transferred to low-resource languages. To this end, both [Chen et al. (2021)](references.html#chen2021mixmatch) and [Casanova et al. (2022)](references.html#casanova2022singlespeaker) us LE (language embeddings), where the former relies on disentangling language and speaker information using adversarial classification, while the latter relies on SCL (speaker consistency loss).

### Differing Target Representations

Another way to adapt TTS architectures for ASR is to predict a target representation other than raw audio. As long as this representation can be derived from raw audio, ASR models can be trained on it. [Chen et al. (2021)](references.html#chen2021mixmatch) do this by doing DTM (direct-to-mel) prediction, predicting mel-spectrograms without using a vocoder. [Hayashi et al. (2018)](references.html#hayashi2018dth) and [Ueno et al. (2021)](references.html#ueno2021dth) predict use DTH (direct-to-hidden) to predict the hidden representations of an ASR model, where the former use their own model and the latter use discrete IDs derived from vq-wav2vec [(Baevski et al., 2019)](references.html#baevski2019vqwav2vec). The advantage of this is that discrete IDs are easier to predict than continuos values, reducing the mismatch between synthetic and real data.

### Sampling Techniques

Since it's essentially free to generate more data once a model is trained, data can be overgenerated and a subset more suitable for **TTS-to-ASR** can be sampled. [Hu et al. (2022)](references.html#hu2022synt) use RS (rejection sampling), where they use the score of an external discriminator to reject samples least likely to be classified as real speech. In my own work, I artificially increase diversity (ID in {numref}`ttstable`) by sampling with more extreme variation information than in the original data. I explain this in detail in the [conditioning chapter](09_conditioning).

In the [next chapter](03_low_resource_asr), I show where and how TTS-generated data is used for low-resource ASR.

[^VAE]: As you can see in {numref}`ttstable`, they are used by [Rosenberg et al. (2019)](references.html#rosenberg2019ttsasr), [Laptev et al. (2020)](references.html#laptev2020moredata), [Sun et al. (2020)](references.html#sun2020vae), [Du & Yu (2020)](references.html#duyu2020sc), [Chen et al. (2021)](references.html#chen2021mixmatch) and [Casanova et al. (2022)](references.html#casanova2022singlespeaker). 