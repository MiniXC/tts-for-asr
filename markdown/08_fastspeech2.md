# FastSpeech 2

My TTS architecture of choice for **TTS-for-ASR** is FastSpeech 2. It non-autoregressive, which sets it apart from the most commonly used architecture for **TTS-for-ASR**, Tacotron 2 [(Shen et al., 2018)](references.html#shen2018tacotron2), and is controllable, as opposed to the less used VITS [(Kim et al., 2021)](references.html#kim2021vits).

```{figure} ../figures/ttscomparison.svg
---
figclass: boxed
name: speakers
---
Comparsion of the three architectures used for **TTS-for-ASR**.
```

```{figure} ../figures/fastspeech2base.svg
---
figclass: boxed
name: speakers
---
An overview of the FastSpeech 2 architecture.
```