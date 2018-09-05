---
layout: post
title: "Language Classifier: Training a simple ANN on English, German and Russian speech"
date: 2018-08-22
--- 

This project served as a wonderful playground for speech data collection and preprocessing as well as getting a 'deep'er understanding of how to apply deep learning neural networks. To see where the repository is currently, click <a href = "https://github.com/a-n-rose/language-classifier">here.</a>

My main goal was to build a classifier that could identify the type of language spoken. To start out, I decided to keep it simple, training a simple artificial neural network (ANN) on only two languages, English and German. That seemed to me a good starting off point.

I collected a lot of English and German (later Russian as well) speech from <a href="http://voxforge.org/">VoxForge</a> and then extracted <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">MFCCs</a> after I added various levels of backgound noise to the speech. I'm glad I did that too! Little did I know but those various levels of noise would make very clear how important noise is in machine learning contexts. (For more on how I handle noise beyond machine learning contexts, read <a href="https://a-n-rose.github.io/2018/08/23/noise-how-i-love-you.html">here</a>.)

## Noise Background

Before I continue, I'll quickly mention some noise background (haha - that wasn't intentional, actually) and the noise techniques I used in this little experiment. 

When adding noise to speech data for machine learning, you can add 'matched' noise, which is noise that is similar to the noise a model will encounter in-action, or random noise, noise that spans a wide range, as you don't know what kind of noise a model will encounter. Maybe someone will record speech next to a construction site and need your model for it.

Since I knew I would test my models with speech recorded by my own computer in a quiet environment, I just used a recording made by my own computer of my quiet aparment. Note: with my comp's horrible mic, the noise signal was actually quite loud. Long story short, I applied matched noise to the speech data.

Before I added the noise signal to the speech signal (i.e. a speaker's recording), I multiplied the noise signal with a random value from the following: 0, 0.25, 0.5, 0.75, 1.0, and 1.25. This either cancelled out, reduced, maintained, or enhanced the orignal noise level, similar to real life, which has many levels of noise present. Once the noise was added, I extracted the MFCCs (with windows of 25ms and window shifts of 10ms).

## Training: Binary Class Classifier (English and German)

The first neural network I trained was a simple ANN, with only 3 layers (including the input and output layers) just to get the hang of it. 

##### Figure 1: Artificial Neural Network (ANN) Illustration
![Imgur](https://i.imgur.com/pfAsfyO.png)
##### Inputs = 40 MFCCs; Outputs = English or German 

I fed it varying numbers of rows of MFCC data, starting with 1 million and working up to 4 million (not to take up too much memory on the machine I used), and also playing with batchsizes and epochs. Eventually I found a sweetspot using batchsizes of 100 and epochs of 50 to show the general nature of how a model trained without taking too long. For reference, a 1 second clip of speech would result in appx. 100 rows of MFCCs. 

Without touching the data or the model, I didn't notice a difference in model performance with the increase of data (because of that I stuck with 2 million rows for each training session). What I did see, was that the model favored one language over the other at a suspiciously consistent rate...(see Figure 3). It classified speech as English nearly twice as often as German, even though they were equally represented. 

Just for kicks, I removed the 1st MFCC to see if that had an effect, as that pertains to amplitude. Nada.

When I compared how well the ANN trained with varying levels of noise, that's when the differences became prominent.

#### Figure 2: Model Accuracy across Noise Condition on Test Dataset
![Imgur](https://i.imgur.com/uwYEPik.png)
##### Accuracy of models trained with no noise ('None: 0'); with varying levels of noise, ranging from 0.25 to 1.25; and a mix of all levels of noise ranging from 0 to 1.25 ('All Levels').

The first model, trained with no noise (i.e. 'None: 0'), showed a classic case of over-fitting. It classified the test set (different from the training set used to train the model; for further information see <a href="https://en.wikipedia.org/wiki/Training,_test,_and_validation_sets">here.</a>) with almost 100% accuracy. It fit its model a bit too closely to the provided speech data and won't likely generalize to other speech data well. The other models did not reach very high accuracy, especially with higher levels of noise, and showed a bias towards English (see below: the models with higher levels of red show bias towards English).

#### Figure 3: Model Classification Error on Test Dataset 
![Imgur](https://i.imgur.com/qHO9YT6.png)
##### Models trained with more noise tended to classify speech as English

I wonder if this is because more of the sounds generated in English are also present in German than the other way around (i.e. more of German's sounds would be considered phonologically illegal in English, like 'pf' in 'Pferd', which means horse by the way). If this is correct, the model used MFCC values not present in English as identifiers of German. I'm just speculating here.

## Applying the Models to Real-World Speech

To see how these models classified newly recorded speech, I created an application to do just that: collect speech and categorize it using these 7 models.

The two people I had at hand to use this application happened to be a native English speaker and a native German speaker (me and my hub-dubs.. er husband, respectively). I was not surprised, with models biased towards English, that all of the models identified my speech as English. Success! Unfortunately, only three of the models identified my hubby's as German. But that's actually great news as that tells me which models are doing a better job.

The models that categorized my husband's speech as German: 
* Low (0.5)
* Medium (0.75)
* All Levels 

I peaked a bit further into the accuracy and confusion matrixes/matrices of each of these models on my newly collected speech. What I found interesting was, even though the 'All Levels' model showed bias towards English with the test dataset, for my brand-new speech, that model showed no bias at all. 

#### Figure 4: Model Classification Error on Newly Recorded Speech
![Imgur](https://i.imgur.com/EPEQkHc.png)
##### The model trained with no noise increased false classifications dramatically while the other models decreased or lost their language bias when applied to new speech. 

Comparing accuracy of the models across test dataset and new speech contexts, it seems the model trained on speech with 'All Levels' of noise performed the best, with 'Very Low', 'Low', and 'Medium' models close on its heals. Considering these numbers, it makes sense the models 'Low', 'Medium' and 'All Levels' identified my husband's speech correctly.

#### Figure 5
![Imgur](https://i.imgur.com/LfDcs7z.png)
##### When applied to brand-new speech, the 'All levels' model achieved the highest accuracy, 59.29%

## Training: Multiple Class Classifier

Just for fun, I added Russian to the mix (after adjusting the classifier to accommodate multiple classes). The accuracy dropped to 50%. Soooooooo, it's time to upgrade the model.

## Next Steps

It will be interesting to see if using Kera's TimeDistributed module reveals a succession of MFCC data to be useful in identifying languages (I suspect yes, as that could allow the models to identify phonological patterns). Eventually I also want to apply random background noises to speech training data, not only matched background noise. 

For now, this little experiment highlights the importance of considering background noise when training, testing, and deploying speech/audio related models. No use of background noise is probably a bad idea, and too much probably won't do any good. Like with so many things, moderation is key.




Articles I've found interesting/helpful:
* For background on <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">MFCCs</a>

* <a href = "https://iamtrask.github.io/2015/07/27/python-network-part2/">Gradient Descent</a> - I found the graphics helpful and made me look at neural nets a new way.

* <a href="https://www.kaggle.com/c/tensorflow-speech-recognition-challenge/discussion/46945">Tips on training models with speech data</a>

* <a href="https://www.analyticsindiamag.com/using-deep-learning-for-sound-classification-an-in-depth-analysis/">Deep learning and sound classification</a>

* A couple articles on audio data prepping for machine/deep learning <a href="https://www.kaggle.com/fizzbuzz/beginner-s-guide-to-audio-data">here</a> and <a href="https://www.analyticsvidhya.com/blog/2017/08/audio-voice-processing-deep-learning/">here</a>
