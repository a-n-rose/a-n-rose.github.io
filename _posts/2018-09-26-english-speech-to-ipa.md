---
layout: post
title: "Training an LSTM on non-aligned annotation and speech data"
date: 2018-09-28
---

Voxforge hosts several large, publicly available speech databases for multiple languages. Included in their speech databases are annotation files associated with individual recordings. I had already downloaded several speech databases in aims of exploring how to <a href = "https://a-n-rose.github.io/2018/08/22/language-classifier.html">train neural networks</a> on <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">MFCC data</a>. Therefore, I decided to see how I could apply the annotations to the training data for an LSTM neural network. Could I build a speech to text model?

Generally speaking, I would like to see how speech recognition models can be built without aligned annotations, meaning annotations that do not exactly correspond with the timing of speech sounds. There are models built on either relatively small amounts of professionally annotated speech data and also on large amounts (i.e. big data) of unannotated speech data. Both have their role in model development and can be used together to build more complex and telling models (<a href="https://www.icsi.berkeley.edu/icsi/blog/data-versus-experts">this blog</> explores this topic in regards to text analysis; this paper by <a href="https://www.semanticscholar.org/paper/Speech-Analysis-in-the-Big-Data-Era-Schuller/3c9567ef956e5ea1a0a20843aea09489d117f5ac">Schuller, 2015</a> reviews data available today, and ways to best analyze them.) 

I would like to know if speech data with approximate annotations offers any sort of contribution to the analysis of speech. If so, this may improve the effectiveness of some speech data, even though the speech data is not professionally annotated. This was the purpose of my experiment with Voxforge's English speech database. While I would like to built multilinguistic models, I explored first if I could develop working models with one language. Click <a href="https://github.com/a-n-rose/language-classifier/tree/master/english_speech_to_ipa">here</a> for the repository.

### Step one: prepare the data

The first step in collecting the data I needed was to understand the <a = href="https://a-n-rose.github.io/2018/09/12/multiling-speech2text.html">architecture</a> of the database(s), specifically where the annotations and corresponding wave files were stored. Once I determined a pattern, I wrote a scipt that unzipped each zip-file individually, extracted the data I needed, and then deleted the unzipped contents. I stored the annotations and the MFCC data in the same database but in different tables. I used the zipfile and wave file names as keys to match the right annotation with the right set of <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">MFCC</a> values.


### Resources:

Schuller, B.W. (2015). Speech Analysis in the Big Data Era. TSD.


