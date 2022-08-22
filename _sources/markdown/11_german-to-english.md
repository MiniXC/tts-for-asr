# German-to-English Transfer

In some of my earliest experiments, I tested a naive **TTS-for-ASR** approach utilizing phone-based multi-lingual transfer from the German subset of the GlobalPhone dataset [(Schultz et al., 2002)](references.html#schultz2002globalphone) to LibriSpeech [(Panayotov et al., 2015)](references.html##panayotov2015librispeech).
To allow for a qualitative assessment of the synthetic speech, I went with an artificial low-resource scenario, using English as the target language, but restricting the amount of data.
I trained the TTS model described in the [FastSpeech 2 chapter](08_fastspeech2) on the German portion of GlobalPhone alone, without any fine-tuning on English data.

As can be seen in {numref}`phone_mapping`, when using the model to generate additional English training data for ASR, this happens in the following steps:
1) Convert English text to phoneme sequence using G2P.
2) Map the phonemes that are only present in English, but not in German, to their closest German counterparts.
3) Synthesize the altered phoneme sequence.

```{figure} ../figures/phone_mapping.png
---
figclass: boxed
name: phone_mapping
height: 200px
---
Phone mapping approach for cross-lingual TTS.
```

## Phone Mapping

Since different languages often have differing phone inventories, it is necessary to map to closest phone in a target language when using this approach. To do so without the need for handcrafting this mapping, I utilized the PHOIBLE database and phonological distinctive feature system [(Moran et al., 2019)](references.html#moran2019phoible), in which every phoneme in every language in said database has a corresponding feature vector. By computing the cosine similarity between phone vectors across different languages, we can substitute non-existing phones in the target language with its closest counterpart in the source language. In the case of German-to-English, for example, the ``ð`` is converted to the closest phoneme in German, ``d``.

`````{note}
:class: tip
I have created an utility library for doing this phone mapping called [phones](https://cdminix.me/phones).
`````

You can hear the result of this approach by listening to the audio below -- A male voice says "This is a test." with a noticeably German accent stemming from the replacement of ``ð`` with ``d``.

<audio controls="1" controlslist="nodownload nofullscreen noremoteplayback" alt="The Sound File">
<source src="https://sndup.net/jf9h/d", type="audio/wav">
Your browser does not support the audio tag.
</audio>

## ASR Training

As mentioned before, I restricted the amount of English data to emulate a low-resource setting, to 1 and 15 hours respectively. I used 15 hours of synthetic speech for all experiments.
I used the large pre-trained XLSR-53 model [(Conneau et al., 2020)](references.html#conneau2020xlsr53) for ASR. I found that for this model, combining the synthetic and real speech yielded worse results. Instead, a pre-training/fine-tuning approach has to be used, where the model is pre-trained on the synthetic data and fine-tuned on the real data. This improved WER by 18% (relative) for the 15 hours setting and by 23% for the 1 hour setting, as can be seen in {numref}`gerenresults`. 


```{figure} ../figures/gerenresults.png
---
figclass: boxed
name: gerenresults
height: 100px
---
WER (CER) for **TTS-for-ASR** using a German TTS system with phone mapping to synthesize English speech.
```

In the [next chapter](12_future), I discuss how I want to continue and expand on this work and expand on it going forward.