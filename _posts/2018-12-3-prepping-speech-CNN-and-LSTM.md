---
layout: post
title: "Replication Study: Prepping Speech for ConvNet and LSTM"
date: 2018-12-03
---


## Current Goal:

Implement similar methodology as <a href="https://www.researchgate.net/publication/327350843_Dysarthric_Speech_Recognition_Using_Convolutional_LSTM_Neural_Network">Kim, M., Cao, B., An, K., and Wang, J. (2018)</a> in their training of a convolutional neural network (CNN) paired with a long-short-term-memory neural network (LSTM). They classified speech as either dysarthric or healthy.

I would like to know if a combined CNN and LSTM work better as a pathological speech identifier, with limited speech data, than the Convolutional Deep Belief Network used by <a href="https://strathprints.strath.ac.uk/64290/">Wu, H., Soraghan, J., Lowit, A., and Caterina, G. D. (2018)</a> to identify general pathological speech (healthy vs. multiple pathologies), or the feature selection method implemented by <a href="https://www.sciencedirect.com/science/article/pii/S1877050917321956">Teixeira, J. P., Fernandes, P. O., and Alves, N. (2017)</a>. 

Both of the latter studies used speech data from the openly available <a href="http://www.stimmdatenbank.coli.uni-saarland.de/index.php4#target">Saarbrücken Voice Database</a> (SVD), which I will also use to better compare my findings.

## Speech Collection:

For the first experiment, I will train and test my model to classify just one pathological speech type: dysphonia. I used only speech that was solely labeled by dysphonia; some speech data contained labels of both dysphonia and laryngitis. Such data samples I left out.

I calculated the log mel-frequency energy features according to the methodologies implemented by Kim, Cao, An, and Wang (2018).

```
import librosa

y,sr = librosa.load("dysphonia_speech.wav")

mel_spec = librosa.feature.melspectrogram(y,sr=sr,hop_length=int(0.010*sr),n_fft=int(0.025*sr))
#first derivative = delta (rate of change)
mel_spec_delta = librosa.feature.delta(mel_spec)

#second derivative = delta delta (acceleration changes)
mel_spec_delta_delta = librosa.feature.delta(mel_spec,order=2)

```

As an example, below are mel spectrograms, as well as the delta/rate of change and delta delta/ acceleration of change take from the speech of a dysphonic speaker as well as a healthy speaker. 


<a target="_blank" href="https://imageshack.com/i/poxBnUlVp"><img src="https://imagizer.imageshack.com/v2/150x100q90/924/xBnUlV.png" border="0">Healthy Male Speaker</a> 

<a target="_blank" href="https://imageshack.com/i/plUYP3qgp"><img src="https://imagizer.imageshack.com/v2/150x100q90/921/UYP3qg.png" border="0">Male Speaker with Dypshonia</a>

Note: the links will turn into pictures once imgur is up and running again. 

## CNN and LSTM

Plan: I will feed these features with a context window frame of 9, first to a CNN then to LSTM. 

Work-in-progress.
