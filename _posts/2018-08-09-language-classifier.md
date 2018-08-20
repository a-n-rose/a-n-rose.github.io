---
layout: post
title: "Language Classifier: Using public speech databases to train deep neural networks to ID types of language spoken"
date: 2018-08-09
--- 

This project serves as a wonderful playground for data collection, data cleaning, and seeing how deep learning works, from outside the black box, of course. I've collected a lot of English and German speech from <a href="http://voxforge.org/">VoxForge</a> and then extracted MFCCs after I added various levels of backgound noise to the speech. 

The first neural network I trained was a simple ANN, with only 3 layers (including the input and output layers) just to get the hang of it. 

![Imgur](https://i.imgur.com/pfAsfyO.png)
##### Inputs = 40 MFCCs; Outputs = English or German 

I fed it varying numbers of rows of MFCC data, starting with 1 million and working up to 4 million, and also playing with batchsizes and epochs. Eventually I found a sweetspot using batchsizes of 100 and epochs of 50 to show the general nature of how a model trained without taking too long. Without touching the data or the model, I didn't notice a difference in model performance with the increase of data. I see though that the model favors one language over the other at a suspiciously consistent rate... It classified speech as English twice as often as German, even though they are equally represented. At least they should be... Just for kicks, I removed the 1st MFCC to see if that had an effect, as that pertains to amplitude, with 2 million rows of data to train: hmmm... not much of a difference.

When I compared how well the ANN trained with varying levels of noise, that's when the differences became prominent.

![Imgur](https://i.imgur.com/Y7KnwpF.png)
#### The graph on the left is likely over-fitting and those with more noise, might not be training well at all.

![Imgur](https://i.imgur.com/PGwsz1U.png)
#### Models trained with more noise tend to classify English; this could be due to German having more unique sounds than English, and the model using those unque sounds as identifiers of German. 

The first model, trained with no noise, shows a classic case of over-fitting. I look forward to testing out how well all of these models classify new speech, not included in the training or test datasets. This is next on the agenda. A little further down the line I'd like to see how CNN models do.   



Articles I've found interesting/helpful:
* <a href = "https://iamtrask.github.io/2015/07/27/python-network-part2/">Gradient Descent</a> - I found the graphics helpful and made me look at neural nets a new way.

* <a href="https://www.kaggle.com/c/tensorflow-speech-recognition-challenge/discussion/46945">Tips on training models with speech data</a>

* <a href="https://www.analyticsindiamag.com/using-deep-learning-for-sound-classification-an-in-depth-analysis/">Deep learning and sound classification</a>

* A couple articles on audio data prepping for machine/deep learning <a href="https://www.kaggle.com/fizzbuzz/beginner-s-guide-to-audio-data">here</a> and <a href="https://www.analyticsvidhya.com/blog/2017/08/audio-voice-processing-deep-learning/">here</a>
