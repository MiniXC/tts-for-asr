# Hybrid DNN-HMM

For my **TTS-for-ASR** experiments, I used *Hybrid HMM-DNN system* due to it's suitability for low-resource ASR I explained in the [low-resource ASR chapter](03_low_resource_asr). So far, I have mostly focused on enhancing TTS systems for better use with ASR systems, I have found some improvements which make ASR more suitable for being trained with TTS data.

## CNN-TDNN

I found that the CNN+TDNN model in Kaldi [^cnntdnn] improves performance over a TDNN for synthetic speech (1.8% relative WER reduction), while not showing any significant difference over a regular TDNN for real speech.

## CMVN

Cepstral mean-variance normalization (CMVN) [(Viikki and Laurila, 1998)](references.html#viikkilaurila1998cmvn), which is a normalization technique linearly transforming the cepstral coefficients to have the same segmental statistics, seems to have a similar effect as a CNN+TDNN in that it only yields a marginal improvement for real speech, but a significant improvement for synthetic speech (3.6% relative WER reduction). 

[^cnntdnn]: [github.com/kaldi-asr/kaldi/blob/master/egs/wsj/s5/local/chain/tuning/run_cnn_tdnn_1a.sh](https://github.com/kaldi-asr/kaldi/blob/master/egs/wsj/s5/local/chain/tuning/run_cnn_tdnn_1a.sh)