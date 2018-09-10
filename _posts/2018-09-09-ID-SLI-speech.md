---
layout: post
title: "Identifying Specific Language Impairment in Children's Speech with LSTM Recurrent Neural Network"
date: 2018-09-09
---

Having tested the waters of neural networks and speech, I just couldn't hold back from playing with the speech I collected from the <a href="https://figshare.com/articles/New_draft_item/2360626">LANNA speech database</a>. If I haven't expressed it thus far, this is what I would really like to be able to do: identify various language pathologies (or other clinical disorders) in children as young as possible, i.e. 3 years and younger. While the speech from this database is of children 4-12 years of age, it is a step in the right direction. 

Let's see how it goes!

I am going to set loose my MFCC extraction scripts and let it label the speech data as either healthy or patient. To read a bit more on how I extract the MFCCs and what that data looks like, check out my <a href="https://a-n-rose.github.io/2018/09/09/MFCC-extraction-prep-speech-4-deep-learning.html">blogpost</a> detailing that. 

This speech data has the children's local environmental noise included and silences have not been removed. I look forward to seeing if the application of my background noise has an effect on how the neural networks learn; whether or not more levels of noise help or hinder the networks, especially since the speech already has background noise present. 

Note: too few data --> too high of accuracy. Can use MFCC to play with various machine learning algorithms though!
