---
layout: post
title: "Comparing Speech Features and Deep Learning Models for Speech Recognition"
date: 2019-02-17
---

This past week I gave a <a href="/2019/01/13/workshop-pyladies-deeplearning-speech.html">workshop</a> exploring speech feature extraction and deep learning models. In this post I will use that repo's code to compare:

1) the feature extraction process

2) training and test accuracy/loss of models

3) the performance of the models on new speech data


I will first train CNN+LSTM models on varying <a href="https://a-n-rose.github.io/2019/02/06/python-train-cnn-lstm-speech-features.html">speech features</a>. Eventually I will compare the CNN and LSTM individually as well. 

I want to answer the following:

**Does adding noise to the training data result in more robust models?**

If so, the models trained with noise should be better at classifying new speech.

**If I implement features used specifically in speech recogntion, will my models better recognize words?**

If the features are useful in speech recognition, and since the models are trained, in this context, for speech recognition, this should be the case. Specifically, by using the 1st and 2nd derivatives, indicating rate of change and rate of acceleration, should result in better classification of words. Also, using the lower filters/coefficients of the FBANK and MFCC features should also help their models' performance.

**Do less filtered data work better with all deep neural networks? Or only with some? (e.g CNN vs LSTM)**

Given that deep neural networks are very good at deciphering which features to use on their own, I assume the least filtered features (i.e. the STFT) should be the most useful features. However, this might not be the case for the LSTM alone: the LSTM has very high computation costs which may mean that it would perform better (on my cpu) with the fewest features, which is also the most filtered: the MFCC 13. 

The dataset used to train the models is the Speech Commands Dataset (2017), available <a href="https://ai.googleblog.com/2017/08/launching-speech-commands-dataset.html">here</a>. To explore the code used to achieve this, please see this <a href="https://github.com/a-n-rose/Build-CNN-or-LSTM-or-CNNLSTM-with-speech-features">repository</a>. 


## Figure 1: Durations of feature extraction and best model performance

| Features | Number of Features  | Noise Added | Delta Added | Feature Extraction Duration (min) | Best Performing Model | Test Acc | Test Loss | Train Duration (min) |
|----:|:----:|:----:|:---:|:---:|:---:|:---:|:---:|:---|
| STFT|201|False|False|25.5|CNN+LSTM|81.7%|0.65|869.0|
| STFT|201|True|False|132.4|CNN+LSTM|72.9%|0.95|678.7|
| FBANK|40|False|False|21.4|CNN+LSTM|60.6%|1.31|268.0|
| FBANK|40|True|False|167.3|CNN+LSTM|57.7%|1.42|147.9|
| FBANK|120|False|True|27.9|CNN+LSTM|66.2%|1.15|395.9|
| FBANK|20|False|False|TBC|TBC|TBC|TBC|TBC|
| FBANK|20|True|False|TBC|TBC|TBC|TBC|TBC|
| FBANK|60|False|True|TBC|TBC|TBC|TBC|TBC|
| FBANK|60|True|True|TBC|TBC|TBC|TBC|TBC|
| MFCC|40|False|False|17.7|CNN+LSTM|11.6%|3.1|58.9|
| MFCC|40|True|False|174.47|CNN+LSTM|15.5%|2.9|55.9|
| MFCC|120|False|True|34.56|TBC|TBC|TBC|TBC|
| MFCC|13|False|False|TBC|TBC|TBC|TBC|TBC|
| MFCC|13|True|False|TBC|TBC|TBC|TBC|TBC|
| MFCC|39|False|True|TBC|TBC|TBC|TBC|TBC|
| MFCC|39|True|True|TBC|TBC|TBC|TBC|TBC|

#### If Delta == True, the features triple in number. The first and second derivatives will be calcuated on the original feautures and added as features. Note: all steps were completed on my CPU machine; on a better machine, the durations may decrease. My computer was not able to compute the delta of the STFT (too much memory)

## Training Accuracy and Loss

![Imgur](https://i.imgur.com/UdA0tnf.png?1)

![Imgur](https://i.imgur.com/wgYwbbY.png?1)
#### The Delta (i.e. 1st and 2nd derivatives) features only applied to features NOT mixed with noise: the addition of noise has not revealed a more robust model, except perhaps for the MFCC features.

![Imgur](https://i.imgur.com/ATayHgw.png?1)

## Figure 2: Model classification of new speech, without and with noise reduction 

| word ~| CNN+LSTM STFT no noise ~| CNN+LSTM STFT with noise ~| CNN+LSTM 40 FBANK with Delta |
|:----|:----:|:----:|:---:|
| bed  | six / cat  | six / six  | six / **bed**  |
| bird | six / bed | right / **bird** |  four / **bird** |
| cat |  right / **cat** | right / **cat** | three / **cat** | 
| dog |  up / **dog** | right / wow | up / wow | 
| down |  three / left |  wow / cat | six / seven |
| eight | cat / **eight** | two / bed | up / bird | 
| house | right / left |  right / seven | happy / seven |
| left | two / **left** |  two / **left** |  six / sheila |
| marvin  | two / **marvin**  |  right / five  |  six / **marvin**  |
| nine  | eight / **nine**  | right / **nine**  |  on / **nine**  |
| no |  two / bed | right / bed |  left / cat |
| on | two / happy | happy / off |  six / cat |
| one | three / two | right / six | six / two |
| right | eight / **right** | wow / **right** | four / **right** |
| sheila | six / **sheila** | wow / **sheila** | six / six |
| six |  **six** / seven  | right / **six**  |  up / seven |
| three  | **three** / seven  | two / seven  |  **three** / stop  |
| tree  |  seven / sheila  | right / sheila  | six / stop  | 
| yes | two / sheila | two / six | on / six |
| zero | two / yes | tree / **zero** | six / **zero** | 


#### labels assigned: 1. without noise reduction / 2. with noise reduction. The models tended to perform better when the speech underwent noise reduction. You can see "six" was assigned quite often as the label without noise reduction. The "s" and "x" sound a bit like white noise, right?

## Summary

While these models didn't do so well with the new speech, I have to give them credit, as the library I used to record the new speech recorded along with it a heck ton of noise. I am working on making those recordings cleaner - maybe it's just on my system, I don't know. To give you an idea of *how much noise* is in these recordings and why noise reduction helped:

![Imgur](https://i.imgur.com/MW6Sm8G.png?1)

That aside, I was surprised to find that the models **not** trained with noise performed better on the new speech. I just tested these models with my own speech and on my own computer, so this finding might not hold up elsewhere. If it *is* the case, however, that noise doesn't necessarily help the CNN+LSTM model, hooray! The addition of noise required much longer feature duration times. It will be interesting to see if this holds true for the CNN and LSTM on their own.

The addition of the 1st and 2nd derivatives did seem to help the FBANK features. I did try to apply them to the STFT features; however, my computer ran out of memory. Just too many features. I'll have to break those further down when saving. That's on my ToDo list.

Lastly, it does seem that the least filtered features work the best, at least for the CNN+LSTM. Even without the 1st and 2nd derivatives, the STFT outperformed the FBANK with the 1st and 2nd derivatives. I'm really curious if those would build an even stronger model!

I am in the process of extracting MFCC 13 and FBANK 20; soon thereafter I will train CNN and LSTM models individually on this data to further explore the strengths and weaknesses of these models. 
