# TTS for Low-Resource ASR

This is a summary of my efforts to generate synthetic speech [^TTS] to improve speech recognition [^ASR] in the first year of my PhD in Speech Recognition at the [CSTR](https://www.cstr.ed.ac.uk/). But wait, can't a model that generates speech only approximate what's already in the data, making it impossible for the speech recognition system to learn anything new?

Yes, but only if we have nothing new to add -- and there are many things we can add, for example new speakers, new lexical content, or even new variances (such as the pitch or duration of the speech). And we can also do this across languages and domains, making a model trained on English speakers produce Swahili speech or a make model trained on audiobooks read medical transcripts!

More data is always better, and the closer the data is to the target domain, the better as well. This means that synthetic data is most useful when there isn't plenty of high-quality data in the target domain available. ASR, in this case, is often described as **low-resource**. A lot of work has been done in this area, with varying definitions of what low-resource means (more on this in [Low-Resource ASR](markdown/03_low_resource_asr)). I will give an overview of how synthetic data has been used to improve low-resource ASR, and present my own findings as well. In {numref}`roadmap`, you can see all the major areas covered in this report and how they are related to each other. 

```{note}
For brevities' sake, I refer to TTS for Low-Resource ASR as **TTS-for-ASR** in this work.
```

```{figure} figures/roadmap.svg
---
figclass: boxed
name: roadmap
---
The main concepts I cover in this report.
```

<!-- There is also a more formal version of this report for submission to the University, which you can download here. -->


[^TTS]: Using a TTS (Text-to-Speech) model.

[^ASR]: Using an ASR (Automatic Speech Recognition) model.