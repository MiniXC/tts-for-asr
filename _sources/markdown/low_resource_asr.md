# Low-Resource ASR

As I have mentioned before, synthetic data is most useful in the case of little data being available in the target domain. ASR for low-resource languages is one of those cases, and the one I have focused my own work on so far. Beyond making ASR models themselves more robust to overfitting, there are two main areas of research in low-resource ASR as described by [Yu et al. (2020)](references.html#yu2020lowresourceoverview): **Data Augmentation** and **Knowledge Transfer**. **Data augmentation** encompasses various techniques which usually add perturbed versions of training samples to the data to increase diversity and make the ASR model more robust. While data augmentation expands the space of the data, **knowledge transfer** is about expanding the space of tasks a model has to perform:
- *Multitask learning* simply adds more tasks (for example, each additional language can be seen as an additional task). When the tasks are closely related, a model trained on many tasks often outperforms a model trained on a single task. One of the reasons for this is that the model learns more robust hidden representations.
- *Transfer learning* aims to transfer model knowledge from a high-resource source task to the low-resource target task, usually using a pre-training/finetuning approach. 
- *Meta learning* means to learn from previous a learning task (also called "learning to learn") to adapt to a new task using meta-data from said initial task.

Fig. 3 shows an overview of these different approaches for low-resource ASR.

```{figure} ../figures/low-resource.svg
---
figclass: boxed
---
Techniques used for low-resource speech recognition.
```

## Synthetic Speech ≠ Data Augmentation
While synthetic speech data is usually referred to as a data augmentation technique, I agree with [Nikolenko (2021, p. 12)](references.html#nikolenko2021synthetic) that while the line between data augmentation and synthetic data can be blurry, it is helpful to distuinguish between the two. To illustrate this, let's assume the most extreme case: *training a model using synthetic data alone* - there is no original data being augmented, so I would hesitate to call this data augmentation.

When viewing synthetic speech as a technique outside of data augmentation, it's also easy to see that many techniques used for **knowledge transfer** in low-resource ASR have been applied to the generation of synthetic speech, making many works in this field a combination of **knowledge transfer** and **data augmentation**. I discuss these in more detail in the [Knowledge Transfer](transfer) and [Data Augmentation](augmentation) chapters.

## TTS & Voice Conversion

I further divide synthetic speech for ASR approaches into **TTS** and **Voice Conversion** approaches.
In both cases the data is *conditionally generated*, as I outlined in the [Synthetic Data chapter](other_fields). The difference between the two is the condition: for **Text**-to-Speech, the main condition is obviously text (although there can be additional conditions), while for **Voice Conversion**, the conditions are an utterance and a desired style, and the result will be an utterance in a different style. While **Voice Conversion** is less popular than **TTS** approaches, it has been successfully used for low-resource ASR by [Thai et al. (2019)](references.html#thai2019improvinglowresource), and there is some work that combines the two by utilizing "Style Equalization" to train a **TTS** model to produce styles based on reference audio without requiring style labels [(Hu et al., 2021)](references.html#hu2022synt). Since I haven't focused on voice conversion so far in my work however, I will give an overview over TTS in the [next chapter](tts).

<!-- TODO: explain how the description of what low-resource is across works (LRSpeech has a good figure on this) -->

<!-- TODO: use rossenbach 2021 to look at how robust different architectures are to low-resource -->