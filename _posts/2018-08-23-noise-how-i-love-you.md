---
layout: post
title: "Noise: How it saved my applications"
date: 2018-08-23
--- 

I never thought I would like noise so much. All it took though, like anything really, was a bit of playing around. 

Two recent projects of mine recorded speech and assessed it somehow, either by comparing it with a target recording (<a href="https://a-n-rose.github.io/2018/08/24/mimic-master-pitchcurve-vs-fingerprint.html">Mimic Master</a>) or by classifying it with an ANN classifier (<a href="https://a-n-rose.github.io/2018/08/22/language-classifier.html">Language Classifier</a>).

Before embarking on these projects, I knew that different cellphones and computers probably had different microphones but I didn't recognize, or never really thought about, how much just those differences alone would influence 'background noise' of a recording. This <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5426841/pdf/sensors-17-00917.pdf">article</a> explores such differences. 

Steps I took to reduce influences of varying microphone standards and actual environmental noise include:
* developed noise subtraction function***
```
def reduce_noise(noise_powerspectrum_mean,noise_powerspectrum_variance, speech_powerspectrum_row,speech_stft_row):
    npm = noise_powerspectrum_mean
    npv = noise_powerspectrum_variance
    spr = speech_powerspectrum_row
    stft_r = speech_stft_row.copy()
    for i in range(len(spr)):
        if spr[i] <= npm[i] + npv[i]:
            stft_r[i] = 1e-3
    return stft_r
```
* developed speech detection functions*
```
def sound_index(rootmeansquare_speech, start = True, rootmeansquare_mean_noise = None):
    if rootmeansquare_mean_noise == None:
        rootmeansquare_mean_noise = 1
    if start == True:
        side = 1
        beg = 0
        end = len(rootmeansquare_speech)
    else:
        side = -1
        beg = len(rootmeansquare_speech)-1
        end = -1
    for row in range(beg,end,side):
        if rootmeansquare_speech[row] > rootmeansquare_mean_noise:
            if suspended_energy(rootmeansquare_speech,row,rootmeansquare_mean_noise,start=start):
                if start==True:
                    #to catch plosive sounds
                    while row >= 0:
                        row -= 1
                        row -= 1
                        if row < 0:
                            row = 0
                        break
                    return row,True
                else:
                    #to catch quiet consonant endings
                    while row <= len(rootmeansquare_speech):
                        row += 1
                        row += 1
                        if row > len(rootmeansquare_speech):
                            row = len(rootmeansquare_speech)
                        break
                    return row,True
    else:
        print("No speech detected.")
    return beg,False
```
* adjusted volume of recorded speech to match target recordings*
```
def matchvol(target_powerspectrum, speech_powerspectrum, speech_stft):
    tmp = np.max(target_powerspectrum)
    smp = np.max(speech_powerspectrum)
    stft = speech_stft.copy()
    if smp > tmp:
        mag = tmp/smp
        stft *= mag
    return stft
```
* removed the very beginning of recordings to remove clicks that might be there*
* scaled speech data with scikit-learn**

###### * Mimic Master 
###### ** Language Classifier
###### *** Both Mimic Master and Language Classifier



To see what the noise reduction and volume matching process look like, here is an example of a 'target sound', in this case a dove, and a user's mimic of that sound. 
##### Figure 1: Audio Signal of Dove
![Imgur](https://i.imgur.com/9JjFU77.png)
##### Figure 2: User's Mimic Before Preprocessing
![Imgur](https://i.imgur.com/B79OTih.png)
##### Note: duration is a little different than the target recording to account for user's delay.
##### Figure 3: User's Mimic Post Noise Reduction
![Imgur](https://i.imgur.com/juexi3F.png)
##### Figure 4: User's Mimic Post Volume Matching
![Imgur](https://i.imgur.com/jp24Gf8.png)
###### The volume was matched with the target recording. As you can see, the volume doesn't match perfectly but this process did aid the program's performance. (Dove recording from <a href="https://freesound.org/">Freesound</a>; visualizations created using <a href="https://www.audacityteam.org/">Adacity</a>)

The noise reduction function also came in handy when I built an application to apply my ANN English German classifier. This application recorded new speech and identified whether it was German or English by applying a trained ANN classifer. Below is a graph of 7 models' performances with either the test dataset (data from VoxForge) or new data (my husband's German and my own English speech). I decided to also see how well the models performed on new speech data that underwent my noise reduction algorithm versus on speech data that didn't. The graph shows that the more robust models performed better on speech that had undergone noise reduction.

##### Figure 5: Accuracy of 7 Classifiers on Different Datasets
![Imgur](https://i.imgur.com/YH9xOAo.png)
##### The models are named based on how much noise was present in their training data. The speech data from the 'New Data' condition is the same except that one went through noise reduction and the other did not. From this experiment, the (more robust) models performed better on the noise reduced speech.

I am working on somehow showing objectively how well the Mimic Game performs based on noise preprocessing... considering I never think it accurately scores my own mimics (I am most definitely a mimic master!) I guess I can't rely on my own judgement. More on that to come!
