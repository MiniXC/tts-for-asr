# Cross-Speaker Transfer

Cross-speaker transfer refers to 

## Sampling

Luong et al. (2019) found simply combining all speakers, even if imbalanced, improved performance.

Yang & He (2020) introduce a sampling strategy that ensures that the probability to draw a specific speaker (or language) ($c_i$) from the training data is $c_i^\alpha$, where alpha can range from 1 (which will have no effect) to 0 (which will lead to uniform sampling from speakers). They found $\alpha=0.2$ to work well.

Cooper et al. (2020) found that creating gender-specific TTS models improved performance.

## Speaker Verification

The approach to pre-train an independent speaker-verification network on thousands of speakers and using the hidden representation of such a network to condition a TTS system was popularized by Jia et al. (2018). Since then, many multi-speaker TTS systems use these pre-trained representations, although in different way:
- *Conditioning* is the most straightforward approach, where the representation is simply added to the input. Cooper et al. (2020) found that adding this information at different locations of the network (e.g. in the encoder, prenet and postnet) yielded the best results.
- *Speaker Consistency Loss* is used by Wang et al. (2020) to enforce d-vectors to be consistent in synthetic and real speech. The idea of a consistency loss comes from a different framework which combines TTS and ASR training, which I discuss in the [speech chain chapter](speechchain).
- *Adversarial Speaker Classification* is introduced by Chen et al. (2021) and aims to make the encoder and VAE component of their network speaker-agnostic to more easily adapt to new speakers.

## Voice Conversion

Huybrechts et al. (2021) successfully used VC (voice conversion) to add synthetic data for speakers with little data to balance the dataset. 


## Zero-Shot and Few-Shot Speaker Adaptation

## Speaker Generation

Stanton et al. (2022) identified a new task which they call speaker generation.

## TTS-for-ASR

