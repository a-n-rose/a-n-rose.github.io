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

Below show the feature type, number of features total, whether or not noise was mixed or delta features were applied and how long the extraction process took. So far I have only tested CNN+LSTM, but of these, you can compare which features resulted in the best models. The best below are the models trained on STFT without noise, STFT with noise, and the FBANK without noise but with delta features. I show below these models' classification of new (and very noisy) speech.

| Features | Number of Features  | Noise Added | Delta Added | Feature Extraction Duration (min) | Best Performing Model | Test Acc | Test Loss | Train Duration (min) |
|----:|:----:|:----:|:---:|:---:|:---:|:---:|:---:|:---|
| STFT|201|False|False|25.5|CNN+LSTM|81.7%|0.65|869.0|
| STFT|201|True|False|132.4|CNN+LSTM|72.9%|0.95|678.7|
| FBANK|40|False|False|21.4|CNN+LSTM|60.6%|1.31|268.0|
| FBANK|40|True|False|167.3|CNN+LSTM|57.7%|1.42|147.9|
| FBANK|120|False|True|27.9|CNN+LSTM|66.2%|1.15|395.9|
| MFCC|40|False|False|17.7|CNN+LSTM|11.6%|3.1|58.9|
| MFCC|40|True|False|174.47|CNN+LSTM|15.5%|2.9|55.9|
| MFCC|120|False|True|34.56|CNN+LSTM|7.7%|3.2|175.54|
| MFCC|13|True|False|179.68|CNN+LSTM|7.7%|3.21|51.292|
| MFCC|39|False|True|23.72 |CNN+LSTM|16.082%|2.86|80.232|


#### If Delta == True, the features triple in number. The first and second derivatives will be calcuated on the original feautures and added as features. Note: all steps were completed on my CPU machine; on a better machine, the durations may decrease. My computer was not able to compute the delta of the STFT (too much memory)

## Training Accuracy and Loss

Below are graphs depicting the training accuracy and loss of the CNN+LSTM with various features. You can see the STFT and FBANK features resulted in quite smooth curves, at least compared with the MFCC features. The best performing model is the one trained on STFT features without mixed noise, followed by the model trained with STFT with noise. The best model trained on FBANK was the one trained including delta features. 

![Imgur](https://i.imgur.com/UdA0tnf.png?1)

![Imgur](https://i.imgur.com/wgYwbbY.png?1)
#### The Delta (i.e. 1st and 2nd derivatives) features only applied to features NOT mixed with noise: the addition of noise has not revealed a more robust model, except perhaps for the MFCC features.

![Imgur](https://i.imgur.com/zyQu98m.png?1)

As the table and graphs above show, not all of the models were worth using. Therefore, instead of wasting my time trying out all of the trained models on new speech, I chose only those above 65% accuracy on the test dataset: the CNN+LSTM trained on STFT without noise, with noise, and the FBANK without noise and with delta features. I chose several words to test the models with: some words all models got correct, some all models got wrong. It is interesting to see how these models categorized the speech, especially incorrectly: which sounds cued the model to categorize the speech the way it did?

## Figure 2: CNN+LSTM Model classification of new speech, without and with noise reduction 

| word ~| STFT no noise ~| STFT with noise ~| 40 FBANK with Delta |
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

Given how much noise was recorded with these speech files, these models did great! I used sounddevice, a library for Python, and I need to fix the settings somehow. Below you will see what the recordings look like before and after noise reduction.

![Imgur](https://i.imgur.com/MW6Sm8G.png?1)

I will rerun these models on cleaner speech soon. But this gives you an idea of how good these models actually are: despite so much interference, they still recognized the correct word (with and without noise reduction) 35-50% of the time. 

That aside, I was surprised to find that the models **not** trained with noise performed better on the new speech. I just tested these models with my own speech and on my own computer, so this finding might not hold up elsewhere. If it *is* the case, however, that noise doesn't necessarily help the CNN+LSTM model, hooray! The addition of noise required much longer feature duration times. It will be interesting to see if this holds true for the CNN and LSTM as separate models.

The addition of the 1st and 2nd derivatives did seem to help the FBANK features. I did try to apply them to the STFT features; however, my computer ran out of memory. Just too many features. I'll have to break those further down when saving. That's on my ToDo list. However, adding the delta features to the MFCC features, both the 13 and 40 coefficients, didn't seem to help much. What I gather, at least for the CNN+LSTM model, MFCC features aren't the best to use, at least when it comes to speech recognition. 

Lastly, it does seem that the least filtered features work the best, at least for the CNN+LSTM. Even without the 1st and 2nd derivatives, the STFT outperformed the FBANK with the 1st and 2nd derivatives. I'm really curious if those would build an even stronger model!

While I still plan on testing these features on CNNs and LSTMs as separate models, as well as exploring if FBANK 20 features (rather than 40) aid or hinder the training of speech recognition models, I think the experiments so far do a pretty good job showing the strengths and weaknesses of training with features used in deep learning and speech recognition, namely:

1) STFT features are ideal, but result in potentially long feature extraction and training times (especially if delta features are included)

2) FBANK features are very good features, especially when combined with delta features

3) MFCC features, at least for speech recognition, are perhaps better left to traditional learning algorithms

4) adding noise to the training data does not necessarily build more robust models; however, the models trained on data with mixed noise did converge faster, resulting in shorter training times. (That said, the feature extraction process when mixing noise is *much* longer. You win some, you lose some, I suppose.)

## Future Work

I would like to apply different kinds of data to this repo code, for example, instead of building a speech recognition model, build a sound classifcation system or a clinical speech disorder detector. I would love to compare which features and models work best for those systems.

Additionally, I would like to expand the type of features collected. The STFT, FBANK, and MFCC are just a few features of many. I hope to add functionality to collect other kinds of features, and also to train other algorithms, such as traditional machine learning algorithms like the support vector machine and random forests. 
