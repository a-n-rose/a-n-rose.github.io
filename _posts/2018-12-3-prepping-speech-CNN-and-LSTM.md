---
layout: post
title: "Replication Study: Prepping Speech for ConvNet and LSTM"
date: 2018-12-03
---


## Current Goal:

I implement similar methodology below as <a href="https://www.researchgate.net/publication/327350843_Dysarthric_Speech_Recognition_Using_Convolutional_LSTM_Neural_Network">Kim, M., Cao, B., An, K., and Wang, J. (2018)</a> in their training of a convolutional neural network (CNN) paired with a long short-term memory neural network (LSTM). They classified speech as either dysarthric or healthy. I applied similar methodology on the classification of healthy vs dysphonic speech, healthy vs clinical (more generally) speech, and male-female speech. The latter two comparisons are in-the-works. Click <a href="https://github.com/a-n-rose/language-classifier/tree/master/clinical_speech/SVD">here</a> for the repository.

I would like to know if a combined CNN and LSTM work better as a pathological speech identifier, with limited speech data, than the Convolutional Deep Belief Network used by <a href="https://strathprints.strath.ac.uk/64290/">Wu, H., Soraghan, J., Lowit, A., and Caterina, G. D. (2018)</a> to identify general pathological speech (healthy vs. multiple pathologies), or the feature selection method implemented by <a href="https://www.sciencedirect.com/science/article/pii/S1877050917321956">Teixeira, J. P., Fernandes, P. O., and Alves, N. (2017)</a> to identify dysphonic or healthy speech. 

Both of the latter two studies used speech data from the openly available <a href="http://www.stimmdatenbank.coli.uni-saarland.de/index.php4#target">Saarbrücken Voice Database</a> (SVD), which I will also use to better compare my findings.

## Speech Collection:

For the first experiment, I trained and tested my model to classify just one pathological speech type: dysphonia. I used only speech that was solely labeled as dysphonic.


| Sex  | Group      | Number of speakers |
| :-------------: |:-------------:| :---:|
| female             | dysphonia      |  41  |
| female            | healthy    | 41  |
| male            | dysphonia    | 39  |
| male            | healthy    |  39  |

I calculated 40 log mel-frequency energy features and their first and second derivatives, according to the methodologies implemented by Kim, Cao, An, and Wang (2018). These values served as the features to train the CNN and LSTM models. 

```
import librosa

y,sr = librosa.load("dysphonia_speech.wav")

mel_spec = librosa.feature.melspectrogram(y,sr=sr,hop_length=int(0.010*sr),n_fft=int(0.025*sr))
#first derivative = delta (rate of change)
mel_spec_delta = librosa.feature.delta(mel_spec)

#second derivative = delta delta (acceleration changes)
mel_spec_delta_delta = librosa.feature.delta(mel_spec,order=2)

```

As an example, below are mel spectrograms, as well as the first derivative (delta/rate of change) and second derivative (delta delta/ acceleration of change) from the speech of a dysphonic speaker as well as a healthy speaker, saying "Hello, how are you?" in German. 

### Healthy Male Speaker
![Imgur](https://i.imgur.com/k8khD7R.png)
#### "Hallo, wie geht es Ihnen?"

### Male Speaker with Dypshonia
![Imgur](https://i.imgur.com/jeVNMEb.png)
#### "Hallo, wie geht es Ihnen?"

## CNN 

I fed the CNN with sections of such spectrograms, with 19 frames in each (context window frame of 9: the 9 frames previous and following a central frame). Each of these frames contain 120 features (the log mel-filterbank energies and the 1st and 2nd derivatives). Finally, CNNs require a dimension indicating color scheme, which is just grayscale in this case. 

Data shape:
(19,120,1)

## LSTM

The LSTM does not need the grayscale component. It does however feed the data as a series. Therefore, the LSTM, on its own, could train with 19 frames in a series, each frame with 120 features.

Data shape:
(19,120)

## CNN + LSTM

Because the data has to go through both of these models, the shape must be prepared differently. First, I fed the data through a CNN, which requires the shape for that above, including the grayscale. 

The LSTM still requires a series of data. This I achieved by requiring each speaker in the dataset to offer the same number of samples. In speech and language, this isn't natural. People talk longer, faster, shorter, etc.; so if I want to use all of the speech data, the samples will be of varying lengths. In this scenario I simply zero-padded the samples of speakers that did not have enough speech data to make up the set number of samples I needed. 

I decided to set the number of samples from each speaker to 608. It is divisable by the frame set size (19), which creates the second dimension you see below: 32. What that means is, each speaker had 32 sets of 19-framed samples, each containing 120 features. In grayscale. 

Data shape:
(num speakers, 32, 19, 120, 1)

## Results (so far)

### Dysphonia vs Healthy Speech

I will expand on this section soon. Briefly, I have found the CNN+LSTM, specifically the time-frequency CNN+LSTM (explained in this <a href="https://www.researchgate.net/publication/327350843_Dysarthric_Speech_Recognition_Using_Convolutional_LSTM_Neural_Network">paper</a>), performed the best.

| Model  | Speech Data      | Accuracy
|: -------------: |:-------------:| 
| TFCNN             | female      |  62.5%  |
| LSTM            | female    | 58.7%  |
| TFCNN+LSTM            | female    | 87.5%  |

Note: all in all, this dataset is small; I trained using only 82 speakers, total (for the female speech). Also, the accuracies were collected with speakers randomly assigned to training, validation, and testing datasets. Accuracy rates change a bit accordingly; however, in the several training and testing sessions I have completed thus far, the accuracies tended to stay around these numbers for the corresponding models. 

First thoughts: This is too small of a dataset. However, the validation loss was quite low while training the TFCNN+LSTM as compared to the TFCNN and LSTM alone. The TFCNN+LSTM also reached quite high accuracy on the test datasets, upon each training session (e.g. after 100 epochs). The paper that also used this data, the one that implemented the convolutional deep belief network, reached an accuracy of 100% for the female speech. My TFCNN+LSTM model achieved accuracies ranging from 80% to 100% for the female speech, with far less time dedicated to feature generation than the paper implementing the convolutional deep belief network. The success of the CNN+LSTM here suggests potential applicability to the identification of clinical speech, especially if more data is available. This I hope to further explore in the following section.

### Clinical vs Healthy Speech

This section is in the works. I collected the six clinical types of speech used in the paper by <a href="https://strathprints.strath.ac.uk/64290/">Wu, H., Soraghan, J., Lowit, A., and Caterina, G. D. (2018)</a>: 1) laryngitis, 2) leukoplakia, 3) Reinke's edema, 4) recurrent laryngeal nerve paralysis, 5) vocal fold carcinoma, 6) vocal fold polyps. I am currently extracting the same features I extracted for the previous experiment: log mel-filterbank energy features and their first and second derivatives.

### Male vs Female Speech

I would like to generate a model that is a bit easier to test in the real-world. Therefore, I will also separate the male and female speech data, of the healthy speakers, and see how well these deep learning models learn and apply to real-world data. Application of noise in training and general speech data augmentation may be explored as well. (I have put together a <a href="https://a-n-rose.github.io/2019/01/13/workshop-pyladies-deeplearning-speech.html">workshop</a> exploring exactly this.)


## Helpful Recources
cnn <a href="http://adventuresinmachinelearning.com/keras-tutorial-cnn-11-lines/">guide</a>

filters <a href="https://adeshpande3.github.io/A-Beginner%27s-Guide-To-Understanding-Convolutional-Neural-Networks-Part-2/">guide</a>

cnn+lstm <a href="https://machinelearningmastery.com/cnn-long-short-term-memory-networks/">guide</a> (especially in the comments section)

