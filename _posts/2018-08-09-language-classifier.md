---
layout: post
title: "Language Classifier: Using public speech databases to train deep neural networks to ID types of language spoken"
date: 2018-08-09
--- 

This project serves as a wonderful playground for data collection, data cleaning, and seeing how deep learning works, from outside the black box, of course. I've collected a lot of English and German speech from <a href="http://voxforge.org/">VoxForge</a> and then extracted MFCCs after I added various levels of backgound to the speech. 

The first neural network I trained was a simple ANN, with only 3 layers (including the input and output layers) just to get the hang of it. I fed it varying numbers of rows, starting with 1 million and working up to 3 million, and also playing with batchsizes and epochs. Eventually I found a sweetspot using batchsizes of 100 and epochs of 50 to show the general nature of how a model trained without taking too long. Without touching the data or the model, I didn't notice a difference in model performance with the increase of data. I see though that the model favors one language over the other at a suspiciously consistent rate... It classified speech as English twice as often as German, even though they are equally represented. At least they should be...

I know I should have done this before playing with the ANN, but it was like Christmas morning - I just couldn't help it, once I had some data to throw at it. After playing a bit, I was in the mood to see what was going on in my dataset. I'm wondering if it has to do with the volume differences of the speech data... I normalized the speech data but perhaps I should have done more with it. For now, a good night's rest is in order. 

To be continued!

Articles I've found interesting/helpful:
* <a href = "https://iamtrask.github.io/2015/07/27/python-network-part2/">Gradient Descent</a> - I found the graphics helpful and made me look at neural nets a new way.

* <a href="https://www.kaggle.com/c/tensorflow-speech-recognition-challenge/discussion/46945">Tips on training models with speech data</a>

* <a href="https://www.kaggle.com/fizzbuzz/beginner-s-guide-to-audio-data">Prepping speech data for analysis</a>
