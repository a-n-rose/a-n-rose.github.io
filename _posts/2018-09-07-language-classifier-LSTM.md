---
layout: post
title: "Upgrading language classifier from a simple artificial neural network to a long short term memory (LSTM) model"
date: 2018-09-07
---

The <a href = "https://a-n-rose.github.io/2018/08/22/language-classifier.html">first time I trained a network on speech data</a>, I trained a 3-layer artificial neural network (ANN) with German and English (and later with Russian) with varying levels of noise. The accuracy rates of the binary classifier were around 67%, and for the multiclass classifier, only 44%. I did not go into that project expecting high accuracy rates. I went in expecting a reference point as I explored more complex networks. Check out the <a href="https://github.com/a-n-rose/language-classifier">repo</a> for this language classifier.

I at first wanted to try out Keras TimeDistributed module but after looking into it, just trying a simple long short term memory recurrent network would be the smarter way to go. (Incorporating TimeDistributed is a step or two beyond that. Baby steps.)

The data features I am using to train the networks are 40 <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">MFCCs</a>. The first ANN was trained on these MFCCs in random order. However, speech sounds do not occur at random. Languages have phonological rules their speakers must adhere to. For example, in English, the sounds *p* and *f* don't exist as a consonant pair, while in German that is a completely normal sound combination: *das Pferd* in German means 'horse'. Therefore, training networks on a series of MFCC data should increase the accuracy rate of identifying a spoken language.

As guidance I referenced Keras's <a href="https://faroit.github.io/keras-docs/1.0.8/">documentation</a>; a <a href="https://machinelearningmastery.com/timedistributed-layer-for-long-short-term-memory-networks-in-python/">tutorial</a> on implementing LSTM via Keras, with and without TimeDistributed; a <a href="https://machinelearningmastery.com/sequence-classification-lstm-recurrent-neural-networks-python-keras/">tutorial</a> on training a sequence classifier via LSTM; and of course Google.

### Reorganizing the MFCC data:

In order to feed the LSTM neural network the MFCCs as a time series, I have to reorganize my data a bit. 

My orignal MFCC data looked something like this:
```
data.shape 
(2000000,40) #(number_samples, number_features)
```
That is 2 million rows of samples, each with 40 MFCCs. In order to be able to feed it to the LSTM, the data needs to look more like this:
```
data.shape
(100000,20,40) #(number_samples, number_in_sequence, number_features)
```
Numpy.reshape() makes such a feat very easy. 

```
import numpy as np

# example set of data: 100 samples, each with 40 features
ex_data_2d = np.random.rand(100,40)

# Now create sequence data
# I want each sequence of MFCC data to contain 20 samples. 
# 100 samples divided by 20 = 5
# my new data has 5 sets of 20 samples (i.e. sequences)
ex_data_3d = ex_data_2d.reshape((5,20,40))
```

Voila! Now I have 5 "samples" each containing a sequnce of MFCC data. You can think of this as instead of training the model with single snap-shots of speech, with really short 'videos' of speech. Ideally this should be able to use phonological rules prevelant in languages to help the classifer in language classification. 

Note: while some of the sequences might combine the MFCCs of two different speech recordings, those will likely be silence and also not numerous enough to influence the learning of the algorithm much. Most of the sequences of MFCCs will belong to one speaker and the specific word they are producing.

#### Stacked LSTM for sequence classification:

Once I prepared my data, it was time to set the parameters of the LSTM model. Using my reference material, I put it together like so:

```
from keras.models import Sequential
from keras.layers import LSTM, Dense
import numpy as np

input_dimension = X_train.shape[2]  #number of features (i.e. 40)
timesteps = 20  
num_labels = 1  # Since I was building a binary classifier, one label was enough (the label could be 0 or 1)

classifier = Sequential()

num_neurons = 100  #in the <a href="https://machinelearningmastery.com/sequence-classification-lstm-recurrent-neural-networks-python-keras/">tutorial</a>, 100 neurons were used.
classifier.add(LSTM(num_neurons, return_sequences=True,
               input_shape=(timesteps, input_dimension)))  
classifier.add(Dense(num_labels, activation='sigmoid'))

#according to Keras's <a href="https://faroit.github.io/keras-docs/1.0.8/optimizers/">documentation</a>, the optimizer 'rmsprop' is good for RNN (LSTM is a type of RNN)
classifier.compile(loss='binary_crossentropy',
              optimizer='rmsprop',
              metrics=['accuracy'])
              
classifier.fit(X_train,y_train,batchsize,epochs)
```
###### Note: for the timesteps, this paper (<a href="https://arxiv.org/pdf/1402.1128.pdf">pdf link</a>) offers 20 as an example, which is why I used 20 here. For the number of neurons, the tutorial I looked at applied 100. So, I tried it out. 

Everything worked with reshaping the data but I got the following error:

```
Error when checking target: expected dense_1 to have 3 dimensions, but got array with shape (80, 1)
```
After googling I found that several others had had similar issues. A <a href = "https://github.com/keras-team/keras/issues/6351">fix</a> they suggested did the trick:

I needed to add a layer before the Dense layer to flatten out the dimensions.
```
classifier.add(Flatten())
```
The first training session revealed promising results: the accuracy on test data increased 10 percentage points!

ANN --> 66.71% 
LSTM --> 76.53%

I trained the LSTM to be a multiple class classifer by adding Russian. Unfortunately the accuracy didn't stay there. Still, it was an improvement when compared with the ANN:

ANN --> 43.59%
LSTM --> 53.15%

### Next Steps:
I intend to play with the LSTM parameters and number of layers as well as explore other ways to improve model performance. 

### Useful resources and articles:
* Tutorial on <a href="https://machinelearningmastery.com/timedistributed-layer-for-long-short-term-memory-networks-in-python/">implementing LSTMs</a> in Python (with and without TimeDistributed)

* Tutorial on <a href="https://machinelearningmastery.com/sequence-classification-lstm-recurrent-neural-networks-python-keras/">implementing sequence classiication LSTM-RNNs</a> in Python

* Tutorial on <a href="https://machinelearningmastery.com/reshape-input-data-long-short-term-memory-networks-keras/">reshaping data</a> for LSTM models.

* <a href="https://www.sciencedirect.com/science/article/pii/S1877050917304544">Paper</a> on development of multilingual speech recognition models.
