# Low-Resource ASR

As I have mentioned before, synthetic data is most useful in the case of little data being available in the target domain. ASR for low-resource languages is one of those cases, and the one I have focused my own work on so far. Beyond making ASR models themselves more robust to overfitting, there are two main areas of research in low-resource ASR as described by [Yu et al. (2020)](references.html#yu2020lowresourceoverview): **Data Augmentation** and **Knowledge Transfer**. **Data augmentation** encompasses various techniques which usually add perturbed versions of training samples to the data to increase diversity and make the ASR model more robust. While data augmentation expands the space of the data, **knowledge transfer** is about expanding the space of tasks a model has to perform:
- *Multitask learning* simply adds more tasks (for example, each additional language can be seen as an additional task). When the tasks are closely related, a model trained on many tasks often outperforms a model trained on a single task. One of the reasons for this is that the model learns more robust hidden representations.
- *Transfer learning* aims to transfer model knowledge from a high-resource source task to the low-resource target task, usually using a pre-training/fine-tuning approach. 
- *Meta learning* means to learn from previous a learning task (also called "learning to learn") to adapt to a new task using meta-data from said initial task.

{numref}`lowresource` shows an overview of these different approaches for low-resource ASR.

```{figure} ../figures/low-resource.svg
---
figclass: boxed
name: lowresource
---
Techniques used for low-resource speech recognition.
```

## Synthetic Speech ≠ Data Augmentation
While synthetic speech data is usually referred to as a data augmentation technique, I agree with [Nikolenko (2021, p. 12)](references.html#nikolenko2021synthetic) that while the line between data augmentation and synthetic data can be blurry, it is helpful to distinguish between the two. To illustrate this, let's assume the most extreme case: *training a model using synthetic data alone* - there is no original data being augmented, so I would hesitate to call this data augmentation.

When viewing synthetic speech as a technique outside of data augmentation, it's also easy to see that many techniques used for **knowledge transfer** in low-resource ASR have been applied to the generation of synthetic speech, making many works in this field a combination of **knowledge transfer** and **data augmentation**. I discuss these in more detail in the [Knowledge Transfer](transfer) and [Data Augmentation](augmentation) chapters.

## TTS & Voice Conversion

I further divide synthetic speech for ASR approaches into **TTS** and **Voice Conversion** approaches.
In both cases the data is *conditionally generated*, as I outlined in the [Synthetic Data chapter](other_fields). The difference between the two is the condition: for **Text**-to-Speech, the main condition is obviously text (although there can be additional conditions), while for **Voice Conversion**, the conditions are an utterance and a desired style, and the result will be an utterance in a different style. While **Voice Conversion** is less popular than **TTS** approaches, it has been successfully used for low-resource ASR by [Thai et al. (2019)](references.html#thai2019improvinglowresource), and there is some work that combines the two by utilizing "Style Equalization" to train a **TTS** model to produce styles based on reference audio without requiring style labels [(Hu et al., 2021)](references.html#hu2022synt).

`````{note}
:class: tip
Since I haven't focused on voice conversion so far in my work, I only gave an overview over TTS in the [synthetic speech chapter](tts).
`````

## What is Low-Resource?

There is no single definition for what is or isn't a low-resource setting for ASR. Further, some works refer to terms like "Moderate Low-Resource" or "Extreme Low-Resource" or "Zero-Resource" (for an example, see {numref}`lrspeech-fig`). There is no consensus on how many hours of supervised data can be considered low-resource. But it's important to highlight that this isn't just about the number of hours of paired data, but also about auxillary data such as a *pronunciation lexicon or model*, *unpaired speech*, *unpaired text* or simply *low-quality paired speech*. For example, a model capable of G2P for 100 languages, many of them considered low-resource, has recently been released by Zhu et al. (2022). It is not obvious to me if a case with paired speech data of just several minutes while having access to a large pronunciation lexicon and unpaired text is less or more of a "low-resource" case than one where an hour of paired speech is available with no unpaired text or pronunciations whatsoever.

`````{admonition} Opinion
:class: tip
While it can be helpful to have labels for how "low-resource" a certain setup is, it is not useful to see this as a purely linear scale from "Rich/High" resource all the way to "Zero" resource.
`````

```{figure} ../figures/lrspeech-table.png
---
figclass: boxed
name: lrspeech-fig
---
Xu et al. (2020) classify both ASR and TTS into "Rich Resource", "Low Resource", "Extremely Low Resource" and "Unsupervised".
```

## Which ASR architectures are best for Low-Resource?

There are two main approaches in recent work for low-resource ASR:
- *Hybrid HMM-DNN systems* trained using LF-MMI (lattice-free maximum mutual information) (Hadian et al., 2018). This is usually achieved using the Kaldi framework [^kaldi], and the DNN portion of the system is most commonly a variant of a TDNN (Time-Delay Neural Network) (Waibel et al., 1989).
- *Large end-to-end models* , which are usually pre-trained on unlabeled data self-supervised learning, and fine-tuned for ASR using CTC (Connectionist Temporal Classification) (Graves & Jaitly, 2014). To some extent, this has been made possible by the efficiency of training Transformers (Vasani et al., 2017) and Conformers, which are more suitable for speech (Guo et al., 2021).

Despite the dominance of the latter for high-resource ASR and other speech-related tasks[^E2E], they are relatively unproven for low-resource ASR -- Alumäe & Kong (2021) found a hybrid CNN-TDNNF model to compare competitively with a fine-tuned XLSR-53 (Conneau et al., 2020) without the need for any pre-training. They also found (unsurprisingly) that a Conformer model without pre-training performed extremely poorly. Perero-Codosero et al. (2022) found that when not doing any pre-training, hybrid models still outperform end-to-end models, even in a relatively high-resource scenario (600 hours of data). Specific to synthetic data augmentation, Rossenbach et al. (2021), found that hybrid models did not benefit from the synthetic data, while Transformer models did. In the field of **TTS-for-ASR**, even recent work utilizing features derived from pre-trained end-to-end models (Ueno et al., 2021) does not fine-tune said models, perhaps due to the large amount of resources required for fine-tuning.

`````{admonition} Opinion
:class: tip
While hybrid models certainly perform well in low-resource settings, pre-trained end-to-end models are currently underutilized in **TTS-for-ASR**.
`````

[^kaldi]: [kaldi.org](https://kaldi-asr.org/)
[^E2E]: As of August 2022, WavLM (Chen et al., 2022), data2vec (Baevski et al., 2022), HuBERT (Hsu et al., 2021), wav2vec 2.0 (Baevski et al., 2020) and other large pre-trained end-to-end models dominate the [SUPERB benchmark leaderboard](https://superbbenchmark.org/leaderboard).

<!-- TODO: use rossenbach 2021 to look at how robust different architectures are to low-resource -->