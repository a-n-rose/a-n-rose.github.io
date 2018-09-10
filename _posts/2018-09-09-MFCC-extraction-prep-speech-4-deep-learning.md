---
layout: post
title: "How I prepare speech for deep learning"
date: 2018-09-09
---

Here I will detail what exactly my speech data preparation process is/was and give a view of what the data looks like. Also, I will detail how the data is processed in each of the neural networks I implement and what kind of results varying speech preparation strategies/neural networks offer.

## The MFCC

For fields examining the speech signal through the lense of deep learning, Mel Frequency Cepstral Coefficients (MFCC) are widely used as representations of the original speech signal. Quickly put, the MFCCs are a translation of the speech signal from the time and amplitude domain to the frequency and power domain, which also reflects humans' perception of those frequencies. For example, humans can distinguish changes in lower frequencies better than in higher frequencies; therefore changes in frequencies (among other things) are not linearly processed, as it pertains to human auditory perception. When MFCCs are calculated, adjustments to the values reflect how human beings would process them. For a great tutorial, click <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">here</a>.

The coefficients applied to deep learning algorithms usually range from 12 (2nd-13th coefficients) to 40 and correspond to the filters applied in the calculations of the MFCC. The lower coefficients reflect the presense of lower frequencies in the signal and the higher coefficients reflect the higher frequencies present. The first coefficient pertains to the loudness of the signal, which is normally not relevant in analysis: the loudness should not determine the speech sounds produced; however, if analysing non-linguistic elements, for example emotion, perhaps loudness would be of interest. Coefficients 2-13 are most relevant for speech recognition, as they pertain to speech sounds. The higher coefficients correlate to background noise and voice quality, which are more relevant for analyses of non-linguistic information, for example, emotion or the state of health a speaker has. 

Note: while MFCCs are applied in many research projects for speech recognition, the <a href="https://www.kaggle.com/c/tensorflow-speech-recognition-challenge/discussion/46945#latest-363147">first place Kaggle contestant</a> for Kaggle's speech recognition contest has stated that they obtained better results using mel-log-spectrum rather than MFCCs. Therefore, I eventually will also explore that avenue at some point. 

## One sample of MFCCs vs a series of samples

One sample of 40 MFCCs is a tiny snapshot of a speech signal, i.e. the frequencies present in just 20-40 ms of speech. The MFCCs of a speech signal are calculated in small shifts, for example 10 ms, in order to catch all variation within the 20-40 ms window. For example, if the calculation windows are 25 ms with 10 ms window shifts (standard), the first set of  40 MFCCs would be calculated on 1-25 ms of speech, the second on the 10-35 ms of speech, the third on the 20-45 ms of speech, etc. Each MFCC calculation reveals 40 coefficients indicating the presense of frequencies in each tiny segment. 

One sample of MFCCs will not reveal enough information to identify which sound is produced; however, it still may be relevant to show what non-linguistic features are present. For example, if millions of MFCC samples are fed to a simple artificial neural network (ANN), which trains on unordered data, there still may result in models useful for identifiying gender, health, emotion of the speaker. This is because such speaker features are constant (in the context of a short speech signal). Usually, a speaker's emotion does not change over a period of milliseconds. 

To identify changes of speech sounds over a period of milliseconds, a *series* of MFCC samples are necessary. If a series of MFCCs are examined, those are useful to infer speech sounds produced and patterns of intonation.

#### Wavefile (time and amplitude domain):
![Imgur](https://i.imgur.com/S7deUnS.png?1)
##### Utterance of 'hippopotamus.' Visualization by Audacity

#### MFCC (frequency and power domain) Sample Series: 
![Imgur](https://i.imgur.com/9rT4FLI.png)
##### Take a look at the patterns evident in this picture. Each 'column' is a snippet of 25 ms of speech from a production of the word 'hippopotamus'. The blue row shows the amplitude (i.e. the first MFCC coefficient). In this row you can speculate which parts of the word were 'voiced' or had higher energy - the darker blue parts ('i','o','otams') and which parts didn't - the whiter parts ('pp','p', possibly 't' and 'm'). The coefficients 2-13 require a bit more study; they are more specifically relevant to the actual sounds rather than their energy levels. And the coefficients 20 upwards reveal a clear pattern that at first glance looks to be stress: secondary stress at 'po' and primary stress at 'pot'. 


## Prepping the Speech Data

A problem in the application of models in the real world is variation in background noise. That can be caused by varying microphone quality, room-size, and of course the amount of noise prevelant wherever the person is recording from. As an example, look at the difference of background noise levels between a sample from Voxforge's German database and a segment of my husband's speech recorded from my computer:

#### Voxforge German speech:
![Imgur](https://i.imgur.com/cuoM85C.png?1)
##### A German female speaker saying 'sowohl im Bezug'.

#### My husband's speech:
![Imgur](https://i.imgur.com/j0B0YHe.png?1)
##### A German male speaker saying 'teilweise weltweit'.

Additionally, one needs to consider how speech data was collected in a database. Voxforge is a collection of speech from international volunteers: people record their speech and upload it. They probably check the speech at some level but I don't know for sure how much background noise is prevelant. Specifically, if I am building a network comparing English and German speech (which I <a href="/2018/08/22/language-classifier.html">did</a>), if the databases are significantly different from each other in their background noises/microphone quality, etc., this might adversely affect the generalizability of a model trained on that data. 

One can go about this problem a few ways. One is to make sure the model is applied to new speech data that is as similar to the training data as possible. Another is to apply 'matched' background noise to the training data. 'Matched' noise is background noise that would be present when the model is applied to new data. If 'matched' noise is applied to the training data, the model should still be sensitive to relevant speech information, despite the presense of noise. And lastly, apply 'random' background noises to the training data. When you have no idea where new speech will be collected, this is probably the best choice. It will likely result in a model that can be applied to speech with a wide variety of background noise. This latter option is on my checklist to try out; for my current project I went the route of adding 'matched' noise to the training data. 

#### 15 seconds of background noise at my apartment:
![Imgur](https://i.imgur.com/piHtRP0.png?1)
##### You can see that this looks a lot more like the recording of my husband than the quiet German sample from Voxforge.

Basically, when I extracted the MFCCs from the Voxforge speech samples, I added random segments from the 15 second background noise wavefile to the speech sample. 

## Language Classifier: ANN vs LSTM

My first attempt at training a neural network classifier on MFCC data was in the context of identifiying the language spoken. I first trained a simple ANN on English and German speech, eventually also Russian, and compared its success with that of a long short term memory (LSTM) recurrent neural network. The former trained on the MFCC at random, and as long as it was a binary classifier (only classifying English vs German), actuallly performed around 60% accuracy. When I built an application to collect new speech and apply the models to that speech, some of the models correctly classified new speech as English or German. The accuracy rate increased 10 percentage points when I trained an LSTM on the MFFC data. 

The biggest difference between the two neural networks is the ANN trains on the MFCC data at random and the LSTM trains on sequences of MFCCs (in my case, 20 MFCC sequences). This allowed the algorithm to identify patterns in the speech productions which logically should help the model identify langauge differences (some speech patterns only exist in one of the three languages).

## Conclusion

The methods I used thus far just scratch the surface of how deep learning can be applied to speech classification. I am primarily interested in how speech (especially child speech) can be used to identify language disorders (or others); however a huge obstacle is the lack of available data. (This is prevalent in my <a href="/2018/09/09/ID-SLI-speech.html">post</a> explore speech data from children with and without specific language impairment) It is not easy to track down thousand upon thousands of reliable, somewhat clean clinical speech data. That said, there is still so much to gain from more prevalent speech data and I still have much to learn about how to tweek models and preprocess the speech data to result in useful findings.



