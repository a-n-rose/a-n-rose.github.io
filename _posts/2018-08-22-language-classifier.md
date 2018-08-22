---
layout: post
title: "Language Classifier: Training a simple ANN on German and English speech"
date: 2018-08-22
--- 

This project served as a wonderful playground for speech data collection and preprocessing as well as getting a 'deep'er understanding of how to apply deep learning neural networks. To see where the repository is currently, click <a href = "https://github.com/a-n-rose/language-classifier">here.</a>

My main goal was to build a classifier that could identify the type of language spoken. To start out, I decided to keep it simple, training a simple ANN on only two languages, English and German. That seemed to me a good starting off point.

I collected a lot of English and German speech from <a href="http://voxforge.org/">VoxForge</a> and then extracted <a href="https://en.wikipedia.org/wiki/Mel-frequency_cepstrum">MFCCs</a> after I added various levels of backgound noise to the speech.

The first neural network I trained was a simple ANN, with only 3 layers (including the input and output layers) just to get the hang of it. 

##### Figure 1: Artificial Neural Network (ANN) Illustration
![Imgur](https://i.imgur.com/pfAsfyO.png)
##### Inputs = 40 MFCCs; Outputs = English or German 

I fed it varying numbers of rows of MFCC data, starting with 1 million and working up to 4 million (not to take up too much memory on the machine I used), and also playing with batchsizes and epochs. Eventually I found a sweetspot using batchsizes of 100 and epochs of 50 to show the general nature of how a model trained without taking too long. 

Without touching the data or the model, I didn't notice a difference in model performance with the increase of data (because of that I stuck with 2 million rows for each training session). What I did see, was that the model favored one language over the other at a suspiciously consistent rate...(see Figure 3) It classified speech as English nearly twice as often as German, even though they were equally represented. 

Just for kicks, I removed the 1st MFCC to see if that had an effect, as that pertains to amplitude. Nada.

When I compared how well the ANN trained with varying levels of noise, that's when the differences became prominent.

#### Figure 2: Model Accuracy and Loss across Noise Condition
![Imgur](https://i.imgur.com/yAA0y1i.png)
##### Accuracy of models trained either with no noise, or with varying levels of noise, ranging from 0.25 (the noise signal was reduced to a quarter of its original amplitude before being added to the speech data) to 1.25 (the noise signal was increased to one and one-quarter of its original amplitude before being added to the speech data). The 'All Levels' model was trained on speech mixed with either no noise or noise at these varying levels.

#### Figure 3
![Imgur](https://i.imgur.com/Qn3FKkh.png)
##### Models trained with more noise tended to classify speech as English

The first model, trained with no noise, showed a classic case of over-fitting. The other models did not reach very high accuracy, especially with higher levels of noise, and showed a bias towards English. I wonder if this is because more of the sounds generated in English are also present in German than the other way around (i.e. more of German's sounds would be considered phonologically illegal in English, like 'pf' in 'Pferd', which means horse by the way). If this is correct, the model used MFCC values not present in English as identifiers of German. I'm just speculating though.

To see how these models classified newly recorded speech, I created an application to do just that: collect speech and categorize it using these 7 models.

The two people I had at hand to use this application happened to be a native English speaker and a native German speaker (me and my hub-dubs.. er husband, respectively). I was not surprised, with models biased towards English, that all of the models identified my speech as English. Success! Unfortunately, only three of the models identified my hubby's as German. But that's actually great news as that tells me which models are doing a better job.

The models that categorized my husband's speech as German: 
* Low (0.5)
* Medium (0.75)
* All Levels 

I peaked a bit further into the accuracy and confusion matrixes/matrices of each of these models on my newly collected speech. What I found interesting was, even though the 'All Levels' model showed bias towards English with the Test Data, for my brand-new speech, that model showed no bias at all. 

#### Figure 4
![Imgur](https://i.imgur.com/tsicvE3.png)
##### Language bias disappears or decreases when models are applied to new speech.

Comparing accuracy of the models across Test Data and New Speech contexts, it seems the model trained on speech with 'All Levels' of noise performed the best, with 'Very Low', 'Low', and 'Medium' models close on its heals. Considering these numbers, it makes sense the models 'Low', 'Medium' and 'All Levels' identified my husband's speech correctly.

#### Figure 5
![Imgur](https://i.imgur.com/LfDcs7z.png)
##### When applied to brand-new speech, the 'All levels' model achieved the highest accuracy, 59.29%


Next steps: It will be interesting to see if using Kera's TimeDistributed module reveals a succession of MFCC data to be useful in identifying languages (I suspect yes, as that could allow the models to identify phonological patterns). I also plan on training these models with additional languages, like Russian, as well as applying random background noises to speech training data, not only matched background noise. (For this experiment I applied background noise of my own environment, recorded by my own computer, since I knew I would record more speech from that same environment.)

For now, this little experiment highlights the importance of considering background noise when training, testing, and deploying speech/audio related models. No use of background noise is probably a bad idea, and too much probably won't do any good. Like with so many things, moderation is key.




Articles I've found interesting/helpful:
* <a href = "https://iamtrask.github.io/2015/07/27/python-network-part2/">Gradient Descent</a> - I found the graphics helpful and made me look at neural nets a new way.

* <a href="https://www.kaggle.com/c/tensorflow-speech-recognition-challenge/discussion/46945">Tips on training models with speech data</a>

* <a href="https://www.analyticsindiamag.com/using-deep-learning-for-sound-classification-an-in-depth-analysis/">Deep learning and sound classification</a>

* A couple articles on audio data prepping for machine/deep learning <a href="https://www.kaggle.com/fizzbuzz/beginner-s-guide-to-audio-data">here</a> and <a href="https://www.analyticsvidhya.com/blog/2017/08/audio-voice-processing-deep-learning/">here</a>
