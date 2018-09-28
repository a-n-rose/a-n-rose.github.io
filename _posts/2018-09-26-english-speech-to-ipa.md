---
layout: post
title: "Training an LSTM on non-aligned annotation and speech data"
date: 2018-09-28
---

Voxforge hosts several large, publicly available speech databases for multiple languages. Included in their speech databases are annotation files associated with individual recordings. I had already downloaded several speech databases in aims of exploring how to <a href = "https://a-n-rose.github.io/2018/08/22/language-classifier.html">train neural networks</a> on <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">MFCC data</a>. Therefore, I decided to see how I could apply the annotations to the training data for an LSTM neural network.

In general, I would like to see how speech recognition models can be built without aligned annotations, meaning annotations that do not correspond with the timing of speech sounds. There are ways to handle this problem, ranging from combining models trained on different speech datasets (sometimes different languages), to including synthesized speech in with the train data. Some techniques are described in this <a href="https://www.semanticscholar.org/paper/Speech-Analysis-in-the-Big-Data-Era-Schuller/3c9567ef956e5ea1a0a20843aea09489d117f5ac">paper</a> by Schuller, 2015. (However there are a number of typos in it..)

Before I delve into such complex models, I wanted experiment with the speech and annotation data I downloaded from Voxforge and see if I could loosely align them to develop a speech to text model. While my goal is to build multilinguistic models, I started out with the English speech data. Click <a href="https://github.com/a-n-rose/language-classifier/tree/master/english_speech_to_ipa">here</a> for the repository.

### Step one: prepare the data

The first step in collecting the data I needed was to understand the <a = href="https://a-n-rose.github.io/2018/09/12/multiling-speech2text.html">architecture</a> of the database(s), specifically where the annotations and corresponding wave files were stored. Once I determined a pattern, I wrote a scipt that unzipped each zip-file individually, extracted the data I needed, and then deleted the unzipped contents. I stored the annotations and the MFCC data in the same database but in different tables. I used the zipfile and wave file names as keys to match the right annotation with the right set of MFCC values.  

Cite:
Schuller, B.W. (2015). Speech Analysis in the Big Data Era. TSD.

### Resources:
* Paper covering speech analysis in the age of <a href="https://www.semanticscholar.org/paper/Speech-Analysis-in-the-Big-Data-Era-Schuller/3c9567ef956e5ea1a0a20843aea09489d117f5ac">big data</a>.
