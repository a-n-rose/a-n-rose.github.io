---
layout: post
title: "Training a Multi-Emotion Classifier with Emotional Speech"
date: 2018-09-11
---

I decided to try out building an emotion classifier and training it with speech data from the <a href="https://zenodo.org/record/1188976">RAVDESS</a> speech database. It will be interesting to see which MFCC coefficients are most relevant. For reference to the code, follow this <a href="https://github.com/a-n-rose/language-classifier/tree/master/emotions_classifier">repository</a>.

I downloaded the audiofiles, which contained the category of emotion in each wavefile's title. 

Filename identifiers 

    Modality (01 = full-AV, 02 = video-only, 03 = audio-only).
    Vocal channel (01 = speech, 02 = song).
    Emotion (01 = neutral, 02 = calm, 03 = happy, 04 = sad, 05 = angry, 06 = fearful, 07 = disgust, 08 = surprised).
    Emotional intensity (01 = normal, 02 = strong). NOTE: There is no strong intensity for the 'neutral' emotion.
    Statement (01 = "Kids are talking by the door", 02 = "Dogs are sitting by the door").
    Repetition (01 = 1st repetition, 02 = 2nd repetition).
    Actor (01 to 24. Odd numbered actors are male, even numbered actors are female).

The code I wrote to label the MFCCs in the database using the above information:

```
for filename in glob.glob('**/*.wav'):
    parts = Path(filename).parts
    #collect label information from filename:
    labels = parts[1]
    modality = int(labels[:2])
    voice_channel = int(labels[3:5])
    emotion = int(labels[6:8])
    intensity = int(labels[9:11])
    statement = int(labels[12:14])
    repetition = int(labels[15:17])
    speaker = int(labels[18:20])
    gender = speaker%2  #0 = female, 1 = male
```

I extracted 40 MFCC feature values from the speech data, with no noise added. The next step will be to feed these to a neural network, for starters the LSTM, as I have had better results with that network in the past. 

Unfortunately, as with <a href="/2018/09/09/ID-SLI-speech.html">another project</a> I'm working on, the one with SLI speech, there is not as much emotion data to train with as I had available for the Language Classifier <a href="/2018/08/22/language-classifier.html">project</a>. In the latter project, I had well over 2 million rows of MFCC samples to use for testing and training; in the current project I only have a quarter of that amount: 534,822.

The first neural network I trained this MFCC data (with all 40 coefficients) on was a 2 layer (technically 3, as I had to apply a 'flatten' layer) LSTM. The resuts made me think the other classifiers I built were actually quite good. The accuracy on the test data was a mere 34.54%, meaning it only classified MFCC samples as the correct emotion 34.54% of the time. I can't say I'm surprised, as there are 8 emotion categories. It is much more difficult to train a network on 8 classes than 2. 

For a bit of reference, I trained all 40 MFCCs on a 3-layer ANN and the accuracy did not worsen too much: acc: 31.84%. How do the accuracy rates change using the first and higher level MFCC coefficients? (To read more about MFCCs and which coefficients are relevant for what analyses, read <a herf="/2018/09/09/MFCC-extraction-prep-speech-4-deep-learning.html">this post</a>.)

When I removed the lower coefficients and included only the 20-40 coefficients, for the ANN, the accuracy dropped even further, to 17.53%. For the LSTM, using only those coefficients brought the accuracy down to 19.25%. If I include the first coefficient with those upper coefficients, accuracy of the ANN goes up to 28.38% and the LSTM accuracy jumps up to 28.12%, basically the same level of accuracy, surprisingly.

So far, it appears the LSTM, using all 40 MFCCs, does better than the ANN. However, this may be because the speakers all said very similar sentences, either 'Kids are talking by the door' or 'Dogs are sitting by the door'; therefore, I will further explore methods used in other research, for example this paper by <a href="https://ieeexplore.ieee.org/abstract/document/7472669/">Trigeorgis et al. (2016)</a>. They apply both an LSTM and CNN to speech in analyzing emotion. Unfortunately, they used speech data from the <a href="https://diuf.unifr.ch/diva/recola/download.html">RECOLA database</a>, which I don't have access to. So, I will make do with the data I have. 

### Papers of interest:
* <a href="https://ieeexplore.ieee.org/abstract/document/7472669/">Adieu features? End-to-end speech emotion recognition using a deep convolutional recurrent network</a>



### Audio files collected from:

"The Ryerson Audio-Visual Database of Emotional Speech and Song (RAVDESS)" by Livingstone & Russo is licensed under CC BY-NA-SC 4.0.
