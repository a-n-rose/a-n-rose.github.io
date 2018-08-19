---
layout: post
title: "Language Classifier: Using public speech databases to train deep neural networks to ID types of language spoken"
date: 2018-08-09
--- 

This project serves as a wonderful playground for data collection, data cleaning, and seeing how deep learning works, from outside the black box, of course. I've collected a lot of English and German speech from <a href="http://voxforge.org/">VoxForge</a> and then extracted MFCCs after I added various levels of backgound noise to the speech. 

The first neural network I trained was a simple ANN, with only 3 layers (including the input and output layers) just to get the hang of it. I fed it varying numbers of rows, starting with 1 million and working up to 4 million, and also playing with batchsizes and epochs. Eventually I found a sweetspot using batchsizes of 100 and epochs of 50 to show the general nature of how a model trained without taking too long. Without touching the data or the model, I didn't notice a difference in model performance with the increase of data. I see though that the model favors one language over the other at a suspiciously consistent rate... It classified speech as English twice as often as German, even though they are equally represented. At least they should be... Just for kicks, I removed the 1st MFCC to see if that had an effect, as that pertains to amplitude, with 2 million rows of data to train: hmmm... not much of a difference.

I know I should have done this before playing with the ANN, but it was like Christmas morning - I just couldn't help it, once I had some data to throw at it. After playing a bit, I was in the mood to see what was going on in my dataset. I'm wondering if it has to do with the volume differences of the speech data... I normalized the speech data but perhaps I should have done more with it. For now, a good night's rest is in order. 

To be continued!

I read through several posts on using MFCCs for training algorithms and not much was done in those cases regarding volume differences. Therefore, I will continue playing with the MFCCs I've extracted, and will eventually move onto CNNs to see if capturing phonemic patterns help. 

Meanwhile, I'm training the model on various levels of 'matched environment noise'. I varied the levels of noise added to the data by multiplying the noise signal with 0, 0.25, 0.5, 0.75, 1.0, or 1.25, at random. I suspect that the trained models with higher amounts of noise would have lower accuracy rates, but higher generalizability, while those trained with lower amounts of noise, particularly no noise, would have higher accuracy rates (with the data at hand) but lower generalizability. 

I suspect that a CNN or RNN might be better suited for this problem. Seeing how MFCCs change over time is relevant in a language's phoneme pattern; that would likely aid in identifying whether a language is German or English, since certain phonemes or combinations of phonemes don't exist in both languages (e.g. 'pf' exists in German but not in English and 'a' in 'apple' exists in English but not in German). Since I also want to examine healthy vs clinical data, it will be interesting to see which type of model would be most relevant (probably depends on the clinical disorder). The next ToDo item on my list: build a CNN or RNN and see how the model does.

An example of the simple ANN visualized with ANNvisualizer 
![Image of ANN](https://imgur.com/pfAsfyO)



Articles I've found interesting/helpful:
* <a href = "https://iamtrask.github.io/2015/07/27/python-network-part2/">Gradient Descent</a> - I found the graphics helpful and made me look at neural nets a new way.

* <a href="https://www.kaggle.com/c/tensorflow-speech-recognition-challenge/discussion/46945">Tips on training models with speech data</a>

* <a href="https://www.kaggle.com/fizzbuzz/beginner-s-guide-to-audio-data">Prepping speech data for analysis</a>

* <a href="https://www.analyticsindiamag.com/using-deep-learning-for-sound-classification-an-in-depth-analysis/">Sound Classification</a>

* <a href="https://www.analyticsvidhya.com/blog/2017/08/audio-voice-processing-deep-learning/">Preparing speech data for data science projects</a>
