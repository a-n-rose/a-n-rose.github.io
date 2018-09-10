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

I am currently extracting 40 MFCC feature values from the speech data, with no noise added. The next step will be to feed to a neural network, for starters the LSTM, as I have had better results with that network in the past. 



### Audio files collected from:

"The Ryerson Audio-Visual Database of Emotional Speech and Song (RAVDESS)" by Livingstone & Russo is licensed under CC BY-NA-SC 4.0.
