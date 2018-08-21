---
layout: post
title: "Language Classifier: Using public speech databases to train deep neural networks to ID types of language spoken"
date: 2018-08-09
--- 

This project serves as a wonderful playground for data collection, data cleaning, and seeing how deep learning works, from outside the black box, of course. To see where the reprository is currently, click <a href = "https://github.com/a-n-rose/language-classifier">here.</a>

I've collected a lot of English and German speech from <a href="http://voxforge.org/">VoxForge</a> and then extracted MFCCs after I added various levels of backgound noise to the speech. 

The first neural network I trained was a simple ANN, with only 3 layers (including the input and output layers) just to get the hang of it. 

![Imgur](https://i.imgur.com/pfAsfyO.png)
##### Inputs = 40 MFCCs; Outputs = English or German 

I fed it varying numbers of rows of MFCC data, starting with 1 million and working up to 4 million, and also playing with batchsizes and epochs. Eventually I found a sweetspot using batchsizes of 100 and epochs of 50 to show the general nature of how a model trained without taking too long. 

Without touching the data or the model, I didn't notice a difference in model performance with the increase of data. I saw, though, that the model favored one language over the other at a suspiciously consistent rate...(see Figure 2) It classified speech as English nearly twice as often as German, even though they are equally represented. 

Just for kicks, I removed the 1st MFCC to see if that had an effect, as that pertains to amplitude. Nada.

When I compared how well the ANN trained with varying levels of noise, that's when the differences became prominent.

###### Figure 1
![Imgur](https://i.imgur.com/yAA0y1i.png)
###### The graph on the left is likely over-fitting and those with more noise, might not be training well at all.

###### Figure 2
![Imgur](https://i.imgur.com/xxaSfBA.png)
###### Models trained with more noise tend to classify English; this could be due to German having more unique sounds than English, and the model using those unque sounds as identifiers of German. 

The first model, trained with no noise, showed a classic case of over-fitting. The other models did not reach high accuracy and showed a bias towards English. I wonder if this is because more of the sounds generated in English are also present in German than the other way around (i.e. more of German's sounds would be considered phonologically illegal in English, like 'pf' in 'Pferd', which means horse).

Before exploring Keras's TimeDistributed module, to see if training with many subsequential MFCCs might help the models identify phonological patterns, I created a little game that collects speech (normalizes it, subtracts noise from it, and extracts MFCCs from it) and deploys each of the models I've created on it. It spits out at the end the language (out of German and English) the model classified it as. 

I am a native speaker of English, so I tried it out, and my husband is a native speaker of German, so I made him try it out too. I was not surprised, with models biased towards English, all of the models (out of 7) almost always identified my speech as English and only around 2-3 of the models identified my hubby's as German. I guess that's something.

I am now peeking a bit further into the accuracy and confusion matrixes/matrices of each of these models on my newly collected English and German (teehee!) speech.


Articles I've found interesting/helpful:
* <a href = "https://iamtrask.github.io/2015/07/27/python-network-part2/">Gradient Descent</a> - I found the graphics helpful and made me look at neural nets a new way.

* <a href="https://www.kaggle.com/c/tensorflow-speech-recognition-challenge/discussion/46945">Tips on training models with speech data</a>

* <a href="https://www.analyticsindiamag.com/using-deep-learning-for-sound-classification-an-in-depth-analysis/">Deep learning and sound classification</a>

* A couple articles on audio data prepping for machine/deep learning <a href="https://www.kaggle.com/fizzbuzz/beginner-s-guide-to-audio-data">here</a> and <a href="https://www.analyticsvidhya.com/blog/2017/08/audio-voice-processing-deep-learning/">here</a>
