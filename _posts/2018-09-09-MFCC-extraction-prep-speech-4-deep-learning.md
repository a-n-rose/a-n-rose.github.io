---
layout: post
title: "How I prepare speech for deep learning"
date: 2018-09-09
---

Here I will detail what exactly my speech data preparation process is/was and give a view of what the data looks like. Also, I will detail how the data is processed in each of the neural networks I implement and what kind of results varying speech preparation strategies/neural networks offer.

## The MFCC

For fields examining the speech signal through the lense of deep learning, Mel Frequency Cepstral Coefficients (MFCC) are widely used as representations of the original speech signal. Quickly put, the MFCCs are a translation of the speech signal from the time and amplitude domain to the frequency and power domain, which also reflects humans' perception of those frequencies. For example, humans can distinguish changes in lower frequencies better than in higher frequencies; therefore changes in frequencies (among other things) are not linearly processed, as it pertains to human auditory perception. When MFCCs are calculated, adjustments to the values reflect how human beings would process them. For a great tutorial, click <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">here</a>.

The coefficients applied to deep learning algorithms usually range from 12 (2nd-13th coefficients) to 40 and correspond to the filters applied in the calculations of the MFCC. The lower coefficients reflect the presence of lower frequencies in the signal and the higher coefficients reflect the higher frequencies present. 

## Relevance of the coefficients

The first coefficient pertains to the loudness of the signal, which is normally not relevant in analysis: the loudness should not determine the speech sounds produced; however, if analysing non-linguistic elements, for example emotion, perhaps loudness would be of interest. Coefficients 2-13 are most relevant for speech recognition, as they pertain to speech sounds. The higher coefficients correlate to background noise and voice quality, which are more relevant for analyses of non-linguistic information, for example, emotion or the state of health a speaker has. 

Note: while MFCCs are applied in many research projects for speech recognition, the <a href="https://www.kaggle.com/c/tensorflow-speech-recognition-challenge/discussion/46945#latest-363147">first place Kaggle contestant</a> for Kaggle's speech recognition contest has stated that they obtained better results using mel-log-spectrum rather than MFCCs. Therefore, I eventually will also explore that avenue at some point. 

## One sample of MFCCs vs a series of samples

One sample of 40 MFCCs is a tiny snapshot of a speech signal, i.e. the frequencies present in just 20-40 ms of speech. The MFCCs of a speech signal are calculated in small shifts, for example 10 ms, in order to catch all variation within the 20-40 ms window. For example, if the calculation windows are 25 ms with 10 ms window shifts (standard), the first set of  40 MFCCs would be calculated on 1-25 ms of speech, the second on the 10-35 ms of speech, the third on the 20-45 ms of speech, etc. Each MFCC calculation reveals 40 coefficients indicating the presence of frequencies in each tiny segment. 

One sample of MFCCs will not reveal enough information to identify which sound is produced; however, it still may be relevant to show what non-linguistic features are present. For example, if millions of MFCC samples are fed to a simple artificial neural network (ANN), which trains on unordered data, there still may result in models useful for identifiying gender, health, emotion of the speaker. This is because such speaker features are constant (in the context of a short speech signal). Usually, a speaker's emotion does not change over a period of milliseconds. 

To identify changes of speech sounds over a period of milliseconds, a *series* of MFCC samples are necessary. If a series of MFCCs are examined, those are useful to infer speech sounds produced and patterns of intonation.

#### Figure 1: Wavefile (time and amplitude domain):
![Imgur](https://i.imgur.com/S7deUnS.png?1)
##### Utterance of 'hippopotamus.' Visualization by Audacity

#### Figure 2: MFCC (frequency and power domain) Sample Series: 
![Imgur](https://i.imgur.com/9rT4FLI.png)
##### Take a look at the patterns evident in this picture. Each 'column' is a snippet of 25 ms of speech from a production of the word 'hippopotamus'. The blue row shows the amplitude (i.e. the first MFCC coefficient). In this row you can speculate which parts of the word were 'voiced' or had higher energy - the darker blue parts ('i','o','otams') and which parts didn't - the whiter parts ('pp','p', possibly 't' and 'm'). The coefficients 2-13 require a bit more study; they are more specifically relevant to the actual sounds rather than their energy levels. And the coefficients 20 upwards reveal a clear pattern that at first glance looks to be stress: secondary stress at 'po' and primary stress at 'pot'. 


## Prepping the Speech Data

I extraced MFCC data depending on the purpose of my model. If I was using the model on new speech, then I would want to train the model in a way to be relevant for that speech (more on that in a second). If I was not applying the model to new speech, rather just building and testing a network based on data already collected, then I might go about extracting the MFCCs differently.


### Just for Analysis

In some projects, for example one in which I build an algorithm trained on speech from children with and without specific language impairment (SLI), I don't have new speech to collect (yet). So, I don't know what potential new speech might sound like or what environmental noise it will contain. Unforunately I also don't know how much background noise there is in all of the collected samples. 

As a result, I have decided to try out two ways: adding no noise to the training data as well as adding the same background noise that I added to my language classifer (see Figure 5). I decided that by adding that background noise, at different levels, that would even-out the presence of noise throughout the dataset. Also, it would likely make the classifer more resilient if more data is added and if the classifier is applied to new data. 

To see the results of my work with that speech so far, read this <a href="/2018/09/09/ID-SLI-speech.html">post</a>.

### Real-World application

For other projects, the goal is to apply the model to newly collected speech, out in the real-world. A problem such a project faces is variation in background noise. Such variation can be caused by a device's microphone quality, the room-size a recording was made, and of course the amount of noise prevelant wherever the person is recording from. 

As an example, look at the difference of background noise levels between a sample from Voxforge's German database (used for training) and a segment of my husband's speech recorded from my computer (used as new data for the model to classify):

#### Figure 3: Voxforge German speech:
![Imgur](https://i.imgur.com/cuoM85C.png?1)
##### A German female speaker saying 'sowohl im Bezug'.

#### Figure 4: My husband's speech:
![Imgur](https://i.imgur.com/j0B0YHe.png?1)
##### A German male speaker saying 'teilweise weltweit'.

You can clearly see that there is much more background noise in the 'real-world' recording.

Additionally, one needs to consider how speech data was collected in a database. Voxforge is a collection of speech from international volunteers: people record their speech and upload it. They probably check the speech at some level but I don't know for sure how much background noise is prevelant. Specifically, if I am building a network comparing English and German speech (which I <a href="/2018/08/22/language-classifier.html">did</a>), if the databases are significantly different from each other in their background noises/microphone quality, etc., this might adversely affect the generalizability of a model trained on that data.

One can go about these problems a few ways. One is to make sure the model is applied to new speech data that is as similar to the training data as possible, and really make sure that background noise is evenly distributed throughout the training data. Another is to apply 'matched' background noise to the training data. 'Matched' noise is background noise that would be present when the model is applied to new data. If 'matched' noise is applied to the training data, the model should still be sensitive to relevant speech information, despite the presence of noise. And lastly, apply 'random' background noises to the training data. When you have no idea where new speech will be collected, this is probably the best choice. It will likely result in a model that can be applied to speech with a wide variety of background noise. This latter option is on my checklist to try out; for the language classifer project I went the route of adding 'matched' noise to the training data as I knew the environment I would apply the model (from my own computer).

#### Figure 5: 15 seconds of background noise at my apartment:
![Imgur](https://i.imgur.com/piHtRP0.png?1)
##### You can see that this looks a lot more like the recording of my husband than the quiet German sample from Voxforge.

To include this noise in with the training data, when I extracted the MFCCs from the Voxforge speech samples, I added random segments from the 15 second background noise wavefile to the speech sample. 

That code looks like this:

First a definition to apply various levels of background noise to the speech sample:

```
def parser(wavefile,num_mfcc,env_noise=None):
    y, sr = librosa.load(wavefile, res_type= 'kaiser_fast')
    y = prep_data.normalize(y)
    
    if env_noise is not None:
        
        #at random apply varying amounts of environment noise
        rand_scale = random.choice([0.0,0.25,0.5,0.75,1.0,1.25])

        #apply environment noise to speech signal
        #have to first match lengths/scale 
        total_length = len(y)/sr
        envnoise_normalized = prep_data.normalize(env_noise)
        envnoise_scaled = prep_data.scale_noise(envnoise_normalized,rand_scale)
        envnoise_matched = prep_data.match_length(envnoise_scaled,sr,total_length)
        if len(envnoise_matched) != len(y):
            diff = int(len(y) - len(envnoise_matched))
            if diff < 0:
                envnoise_matched = envnoise_matched[:diff]
            else:
                envnoise_matched = np.append(envnoise_matched,np.zeros(diff,))
        
        #add noise to speech values
        y += envnoise_matched
        
    # now extract MFCC features
    mfccs = librosa.feature.mfcc(y, sr, n_mfcc=num_mfcc,hop_length=int(0.010*sr),n_fft=int(0.025*sr))
    
    #return the features, sampling rate, and the amount of noise applied
    return mfccs, sr, rand_scale
```
Then applied the function as I collected the speech files:

```
waves_list = []
for w in glob.glob('/tmp/audio/**/*.wav',recursive=True):
    waves_list.append(w)
if len(waves_list) > 0:
    for k in range(len(waves_list)):
        wav = waves_list[k]
        feature,sr,noise_scale = parser(wav, num_mfcc,env_noise)
```

If you notice, 'env_noise' can be 'None'. This was to allow the possibility of not adding background noise to the training data. Regardless, once the features (i.e. MFCCs) were extracted, they were then saved via SQL in a database. For an idea of how many MFCC samples one speech recording had, if a recording was 1 second long, it had appx. 250 samples.  After all of the speech files had been processed, on they went to a neural network.

Note: since the classifer was trained on the sounds present in the speech, not on the content of the speech (i.e. semantics), I did not limit the number of samples per recording. I wanted as many instances of a language's sound included as possible, therefore I did not throw any data away. 

## MFCCs for the ANN vs LSTM

My first attempt at training a neural network classifier on MFCC data was in the context of identifiying the language spoken. I first trained an ANN on English and German speech, eventually also Russian, and compared those models' success with that of models generated from a long short term memory (LSTM) recurrent neural network. The former trained on the MFCC data at random, and as long as it was a binary classifier (only classifying English vs German), some of the models actually performed pretty well. To explore in further detail what I did, check out this <a href="/2018/08/22/language-classifier.html">post</a>. 

Regarding how the MFCC data was processed by each of the networks, for the ANN, each sample of the MFCC data was processed, but not in succession. The 2 million rows were separated into training and testing data: 1,600,000 in training, 400,000 in testing. The language types were equally represented. The training data were applied to the neural network at random. For testing model performance, test MFCC data were used. The neural network had to classify whether the 40 MFCCs belonged to that of an English speaker or German speaker, and eventually also Russian speaker. To test on new data, the classifer had to classify each 40 MFCC sample of the new signal as either German or English; how the ANN labeled most of the MFCC samples determined the label the ANN classified the speech as. 

The MFCC data were similarly applied to the LSTM, however instead of training on the data samples individually and at random, it learned from a series of 20 MFCC samples consecutively. Beyond the 20 sequences, though, the data were trained randomly. So, just like the ANN, 1,600,000 MFCC samples were fed as training data, 400,000 as testing data. You can read more about this project <a href="/2018/09/07/language-classifier-LSTM.html">here</a>. After having seen the patterns prevelant in the MFCC samples in Figure 2, you can probably guess that the LSTM RNN produced a better performing classifer. 

The other project I mentioned, the one using speech from children with and without SLI, has so far not been so successful. There were nowhere close to 2,000,000 rows of data available, rather 161,100 MFCC samples for training and 39,920 for testing data. Additionally, healthy and patient data were not represented evenly: there were more samples of patient than healthy speech. I did find differences in supplying the available data I had to the networks: when I did not add the noise, the LSTM seemed to over-fit the data; when I added noise, the LSTM did not overfit as much. 

## Conclusion

The methods I used thus far just scratch the surface of how deep learning can be applied to speech classification. I am primarily interested in how speech (especially child speech) can be used to identify language disorders (or others); however a huge obstacle is the lack of available data. It is not easy to track down thousand upon thousands of reliable, somewhat clean clinical speech data. That said, there is still so much to gain from more prevalent speech data and I still have much to learn about how to tweek models and preprocess the speech data to result in useful findings.



