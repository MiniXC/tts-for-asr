# Utterance-Level Conditioning

For the following experiments, I chose
LibriTTS [(Zen et al., 2019)](references.html#zen2019libritts)  as our training dataset and LibriSpeech [(Panayotov et al., 2015)](references.html#panayotov2015librispeech) for
validation. When training the TTS system, I always used
the ``train-clean-360`` split, and when training the ASR
system, we always use ``train-clean-100`` to avoid overlap. To make my experiments as close to the case in which
little real data is available, I limited the size of both synthetic
and real speech datasets to 10 hours. The synthetic speech
was always generated using the transcripts found in the corresponding real data to avoid lexical mismatch between TTS
and ASR.

As I discussed in [the TTS chapter](02_tts.html#controllability), FastSpeech 2 is controllable due to it's variances. [Ren et al. (2021)](references.html#ren2021fastspeech2) describe controlling the output by multiplying these variances by a factor, allowing to generate faster/slower, more high-pitched/low-pitched or louder/quieter speech. I investigated if this could be used to make up for the diversity that is lost due to [oversmoothing](02_tts.html#controllability). In my experiments I modified the phone-level utterance predictions created by the model at inference time. I attempted this in the following ways:
- add gaussian noise to the predicted duration
- item sample from all observed durations during training
    - conditioned on the respective speakers
	- conditioned on the respective phones
- scale the predicted durations by a random factor

I call this variance augmentation.
As you can see in {numref}`varaug`, only sampling from observed observations, conditioned on phones seems to yield a (slight) improvement. I hypothesized that the improvement is only slight due to a trade-off between more diverse speech improving ASR performance, while the random nature of the resulting diversity is hindering ASR training.

```{figure} ../figures/varaug.png
---
figclass: boxed
name: varaug
height: 200px
---
Effects of variance augmentation for ASR training.
```

To evaluate if more diverse variances are helpful at all, I conducted a range of oracle experiments which use the ground-truth variances rather than the predicted ones. This can be seen in {numref}`oracles` in terms of $WER_s/WER_r$ (the ratio of WER when trained on synthetic data to WER when trained on real data). 

```{figure} ../figures/oracles.png
---
figclass: boxed
name: oracles
height: 200px
---
The improvement gained from using differing ground-
truth variance values. ”None” uses predicted Variances, while
”All” uses all ground-truth values..
```

## Utterance-Level Variance Control

Instead of altering the synthetic speech at a phone level, I added one input to the model for each variance, representing the variance mean across the utterance. This reduces the one-to-many problem of TTS at the cost of needing to provide a value for each variance at inference. Since the variances are specific to speakers, I created a normal distribution of each variance for each speaker. Two example distributions for energy and duration can be seen in {numref}`vardists`. At inference time, a random sample is drawn from each distribution and fed to the model -- represented by the red line in {numref}`vardists`.

```{figure} ../figures/vardists.png
---
figclass: boxed
name: vardists
height: 150px
---
Effects of variance augmentation for ASR training.
```

However, it turns out sampling these variances from a normal distribution doesn't improve but decreases performance (2.1% relative WER increase). When visualising the original distributions using KDE (Kernel Density Estimation), it becomes clear why this is the case -- the ground-truth distributions are fat-tailed, with many outliers on either side. This means that approximating this with a normal distribution effectively reduces performance. Once I replaced the sampling distribution with a uniform one, **Utterance-Level Conditioning** the TTS model improved ASR performance.