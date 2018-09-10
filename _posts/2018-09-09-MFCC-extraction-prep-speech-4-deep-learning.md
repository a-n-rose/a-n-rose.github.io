---
layout: post
title: "How I prepare speech for deep learning"
date: 2018-09-09
---

Here I will detail what exactly my speech data preparation process is/was and give a view of what the data looks like. Also, I will detail how the data is processed in each of the neural networks I implement and what kind of results varying speech preparation strategies/neural networks offer.

### The MFCC

For fields examining the speech signal through the lense of deep learning, Mel Frequency Cepstral Coefficients (MFCC) are widely used as representations of the original speech signal. Quickly put, the MFCCs translate the speech signal from the time and amplitude domain to the frequency and power domain, which also reflects humans' perception of those frequencies. For example, humans can distinguish changes in lower frequencies better than in higher frequencies; therefore changes in frequencies are not all equal, as it pertains to human auditory perception. When MFCCs are calculated, adjustments to the values reflect how human beings would process them. 

The coefficients usually range from 12 to 40 and correspond to the filters applied in the calculations of the MFCC. The lower coefficients reflect the presense of lower frequencies in the signal and the higher coefficients reflect the higher frequencies present. The first coefficient pertains to the loudness of the signal, which is normally not relevant in analysis: the loudness should not determine the speech sounds produced; however, if analysing non-linguistic elements, for example emotion, perhaps loudness would be of interest. Coefficients 2-13 are most relevant for speech recognition, as they pertain to speech sounds. The higher coefficients correlate to background noise and voice quality, which are more relevant for analyses of non-linguistic information, for example, emotion or the state of health a speaker has. 

Note: while MFCCs are applied in many research projects for speech recognition, the <a href="https://www.kaggle.com/c/tensorflow-speech-recognition-challenge/discussion/46945#latest-363147">first place Kaggle contestant</a> for the Kaggle's speech recognition contest has stated that they obtained better results using mel-log-spectrum rather than MFCCs. Therefore, I eventually will also explore that avenue at some point. For now, due to so much research using 40 MFCCs, particularly that of identifying pathologies via speech, I will stick to that.

### One sample of 40 MFCC vs 20 samples of 40 MFCCs

One sample of 40 MFCCs is a tiny snapshot of a speech signal, i.e. the frequencies present in just 20-40 ms of speech. The MFCCs of a speech signal are calculated in small shifts, for example 10 ms, in order to catch all variation within the 20-40 ms window. For example, if the calculation windows are 25 ms with 10 ms window shifts (standard), the first set of  40 MFCCs would be calculated on 1-25 ms of speech, the second on the 10-35 ms of speech, the third on the 20-45 ms of speech, etc. Each MFCC calculation reveals 40 coefficients indicating the presense of frequencies in each tiny segment. 

One sample of MFCCs will not reveal enough information to identify which sound is produced; however, it still may be relevant to show what non-linguistic features are present. For example, if millions of MFCC samples are fed to a simple artificial neural network (ANN), which trains on unordered data, there still may result in models useful for identifiying gender, health, emotion of the speaker. This is because such speaker features are constant (in the context of a short speech signal). Usually, a speaker's emotion does not change over the process of milliseconds. 

To identify changes of speech sounds over a period of milliseconds, a series of MFCC samples are necessary. If a series of MFCCs are examined, those are useful to infer speech sounds produced and patterns of intonation.

#### 40 MFCC samples 
![Imgur](https://i.imgur.com/9rT4FLI.png)
##### Take a look at the patterns evident in this picture. Each 'column' is a snippet of 25 ms of speech from a production of the word 'hippopotamus'. The blue row shows the amplitude (i.e. the first MFCC coefficient). In this row you can speculate which parts of the word were 'voiced' or had higher energy - the darker blue parts ('i','o','otams') and which parts didn't - the whiter parts ('pp','p', possibly 't'). The coefficients 2-13 require a bit more study; they are more specifically relevant to the actual sounds rather than their energy levels. And the coefficients 20 upwards reveal a clear pattern that at first glance looks to be stress: secondary stress at 'po' and primary stress at 'pot'. 


### Language Classifier: ANN vs LSTM




