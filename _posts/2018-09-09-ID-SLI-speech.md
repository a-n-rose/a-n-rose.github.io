---
layout: post
title: "Identifying Specific Language Impairment in Children's Speech with LSTM Recurrent Neural Network"
date: 2018-09-09
---

Having tested the waters of neural networks and speech, I just couldn't hold back from playing with the speech I collected from the <a href="https://figshare.com/articles/New_draft_item/2360626">LANNA speech database</a>. If I haven't expressed it thus far, this is what I would really like to be able to do: identify various language pathologies (or other clinical disorders) in children as young as possible, i.e. 3 years and younger. While the speech from this database is of children 4-12 years of age, it is a step in the right direction. Btw, here is the <a href="https://github.com/a-n-rose/language-classifier/tree/master/clinical_speech/LANNA_database">repo</a> for that. 

Let's see how it goes!

I am going to set loose my MFCC extraction scripts and let it label the speech data as either healthy or patient. To read a bit more on how I extract the MFCCs and what that data looks like, check out my <a href="https://a-n-rose.github.io/2018/09/09/MFCC-extraction-prep-speech-4-deep-learning.html">blogpost</a> detailing that. 

This speech data has the children's local environmental noise included and silences have not been removed. I will extract the MFCCs twice: once without any background noise added and once with background noise added (the same I added to my language classifer, as I found the classifier trained on speech with noise was more resilient). 

I trained a LSTM with 20 sequences of 40 MFCC features as each sample. For the no noise condition, the accuracy of the model on the test data was very high: 98.82%. For the additional noise condition, accuracy of the model on test data dropped to 86.02%, what means to me to be a significant improvement. 

Quickly, what these accuracy scores show, is the percentage the models identified test samples of MFCC features correctly as either 'Healthy' or 'Patient' speech (i.e. speech from a child with SLI).

It's just a guess here but I'd say the model is overfitting the data. The model seems to decrease in overfitting when noise is added to the training data; however, I doubt it is generalizable. First of all, the data are not numerous enough to create a generalizable model. For the language classifier I had well over 2 million samples of MFCC data to train and test the ANN/LSTM; for this project I have just 201,020 samples of MFCC features. Also, there is significantly more 'Patient' speech than 'Healthy' speech. Therefore, for the time being, I will continue exploring this speech data in other machine learning contexts, to see what can be gathered about the features and what they may indicate for healthy vs clinical speech. 

Just for kicks, I decided to see what would happen when training a LSTM on MFCC features that were extracted from the speech while implementing a voice activity detection function. Well, the accuracy rate of the LSTM trained without added noise rose from 98.82% to 99.60% and the LSTM trained with added nosie rose from 86.02% to 

I know I know.. I need some graphs on here. I'll get to that. At some point. 

