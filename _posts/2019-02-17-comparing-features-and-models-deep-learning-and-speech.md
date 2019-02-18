---
layout: post
title: "Comparing Speech Features and Deep Learning Models for Speech Recognition"
date: 2019-02-17
---


The purpose of this post is to see which kind of speech features are best to apply to deep learning neural networks for the purpose of speech recognition. I compare the extraction of various types of features, with and without added noise, and the success of convolutional neural networks and long short-term memory neural networks trained on that data. I consider time taken to extract the features and to train the models, the test accuracy and loss, as well as how well the models classify new speech, both female and male speech. The speech data used to train are the Speech Commands Dataset (2017), available <a href="https://ai.googleblog.com/2017/08/launching-speech-commands-dataset.html">here</a>. 

## Expectations

I have three expectations:

1) I expect the STFT features to result in the best models.

Deep neural networks work very well when allowed to filter complex features themselves, rather than learn from the features humans filter for them. The FBANK and MFCC are more filtered than the STFT. For more on what I mean, refer to this <a href="https://a-n-rose.github.io/2019/02/06/python-train-cnn-lstm-speech-features.html">post</a> on the speech features used.

2) The models trained on speech mixed with noise will handle new real-world speech better than those not trained with added noise. 

This is a common practice in the training of models: to train them with noise that may very well occur in the real-world. Adding noise keeps the model from over-fitting during training, which is learning too specifically on the data available at training. I only used one form of noise (a file from the Speech Commands Dataset); I plan on expanding on this in future. 

3) The CNN+LSTM models will perform the best and the CNN models will perform slightly worse with much lower computation cost. The LSTMs will have long training times and will not outperform the CNNs in general. 

CNNs are masters at breaking complex data down into the most relevant features. This means that only the most relevant information is used when computing, and also the neurons within the network only communicate with other nearby neurons. This means that the neurons don't have to interact with all other neurons in their layer, further reducing computation. 

While CNNs are good at recognizing patterns, they may miss some patterns that occur in a time series, which means the addition of LSTM layers may help the model catch otherwise missed patterns, resulting in a better model. 

LSTMs, on the other hand, tend to be very computationally expensive. While they are good at identifying patterns in long time-series, it could be that because speech is so complicated, without the CNN to break the features down, the LSTM will not be able to effectively learn from the data. If this is correct, the LSTM alone should perform best on the MFCC features, specifically the 13 MFCCs. These are the least complex features of all implemented here.

## About the Project

To explore the code used to achieve this, please see this <a href="https://github.com/a-n-rose/Build-CNN-or-LSTM-or-CNNLSTM-with-speech-features">repository</a>. For this specific problem, because the data used is prepared for speech recognition, **I will compare features used typically for speech recognition**. These are **1st and 2nd derivatives**, or delta and delta delta, which show rate of change and rate of acceleration. Additionally, **the lower filters/coefficients of the FBANK and MFCC (i.e. 20 FBANK and 13 MFCC) are most relevant for speech sounds**. I will see how these features influence the success of the speech recognition models. I add the noise to test if this makes the model more robust when introduced to real world speech data. This should make models more robust, regardless the purpose of the model.

During training, if a model's validation loss does not improve after 10 consecutive epochs, training will be stopped. The best model throughout training will be saved, as well as the final model achieved. This may result in varying number of training epochs for the models even though they were all set to complete 100 epochs.

## Variables for Feature Extraction and Model Setup

The feature extraction variables I adjust include:

* type of feature (stft, fbank, mfcc)
* number of features (fbank = 20 vs 40; mfcc = 13 vs 40)
* inclusion of 1st and 2nd derivatives 
* mixture of noise 

The models I will compare:
* CNN: 32 feature maps, kernel size (8,4), max pooling pool size (3,3)
* LSTM: same number of cells as the number of features
* CNN+LSTM: combination of the two above


## Figure 1: 

Durations of Feature Extraction and Best Model Performance

| Features | Number of Features  | Noise Added | Delta Added | Feature Extraction Duration (min) | Best Performing Model | Test Acc | Test Loss | Train Duration (min) |
|----:|:----:|:----:|:---:|:---:|:---:|:---:|:---:|:---|
| STFT|201|False|False|25.5|CNN+LSTM|81.7%|0.65|869.0|
| STFT|201|True|False|132.4|CNN+LSTM|72.9%|0.95|678.7|
| STFT|603|False|True|TBC|TBC|TBC|TBC|TBC|
| STFT|603|True|True|TBC|TBC|TBC|TBC|TBC|
| FBANK|40|False|False|21.4|TBC|TBC|TBC|TBC|
| FBANK|40|True|False|167.3|CNN+LSTM|57.7%|1.42|147.9|
| FBANK|120|False|True|TBC|TBC|TBC|TBC|TBC|
| FBANK|120|True|True|TBC|TBC|TBC|TBC|TBC|
| FBANK|20|False|False|TBC|TBC|TBC|TBC|TBC|
| FBANK|20|True|False|TBC|TBC|TBC|TBC|TBC|
| FBANK|60|False|True|TBC|TBC|TBC|TBC|TBC|
| FBANK|60|True|True|TBC|TBC|TBC|TBC|TBC|
| MFCC|40|False|False|17.7|CNN+LSTM|11.6%|3.1|58.9|
| MFCC|40|True|False|174.47|CNN+LSTM|15.5%|2.9|55.9|
| MFCC|120|False|True|TBC|TBC|TBC|TBC|TBC|
| MFCC|120|True|True|TBC|TBC|TBC|TBC|TBC|
| MFCC|13|False|False|TBC|TBC|TBC|TBC|TBC|
| MFCC|13|True|False|TBC|TBC|TBC|TBC|TBC|
| MFCC|39|False|True|TBC|TBC|TBC|TBC|TBC|
| MFCC|39|True|True|TBC|TBC|TBC|TBC|TBC|

#### If Delta == True, the features triple in number. The first and second derivatives will be calcuated on the original feautures and added as features. Note: all steps were completed on my CPU machine; on a better machine, the durations may decrease.

## Graphs

![Imgur](https://i.imgur.com/UdA0tnf.png?1)

## FBANK

#### CNN+LSTM Accuracy trained on 40 FBANK with noise
![Imgur](https://i.imgur.com/c828ORN.png)

#### CNN+LSTM Loss trained on 40 FBANK with noise
![Imgur](https://i.imgur.com/KyvK4LW.png)

![Imgur](https://i.imgur.com/ATayHgw.png?1)


