---
layout: post
title: "Mimic Master: Comparing Audio Signals Using Chromaprint and Pitch Analysis"
date: 2018-08-01
---

Ever wanted to improve your animal mimicking skills? Well, you're in the right place. While this is still a work in progress, you can try out the best working version <a href = "https://github.com/a-n-rose/mimic-master-how-well-can-you-mimic">here.</a>

This has been a blast and adventure to build. I have come across several problems, ranging from differencs in microphone quality to detecting speech amidst noise, let alone comparing pitch or prosody of two audio signals. 

Any app that records a user remotely has to deal with varying microphone quality and background noise levels. My first challenge was finding a simple way to cancel out whatever background noise a user might have due to those variables. 

I attempted to recreate the noise reduction technique Audacity performs very well; I would say I was relatively successful as background noise was removed from the user's mimics, which improved the analysis of their speech. In the game, I started out by testing the user's mic by recording 5 seconds of their background noise. I calculated the spectral power in that recording and subtracted it from all of the user's subsequent mimics. 

##### Functions defined (see <a href="https://github.com/a-n-rose/mimic-master-how-well-can-you-mimic">the repository</a> for the entire code): 
```
import sounddevice as sd
import librosa

def record_user(duration):
    fs = 22050
    user_rec = sd.rec(int(duration*fs),samplerate=fs,channels=1)
    sd.wait()   
    return(user_rec)

def savewave(filename,samples,sr):
    librosa.output.write_wav(filename,samples,sr)
    print("File has been saved")
    return None
```            
##### Put those functions to work to get and save background noise:
```
duration = 5
print("\nThis next step will take just {} seconds\n".format(duration))
background_noise = record_user(duration)
savewave("backgroundnoise.wav",background_noise,sr=22050)

```
##### Collect the user's mimic (after playing the target sound, of course):
```
user_mimic = record_user(duration_of_target_signal)
savewave("user_mimic.wav",user_mimic,sr=22050)

```
##### Calculate the power spectrum of noise and speech after applying <a href="https://en.wikipedia.org/wiki/Short-time_Fourier_transform">Short-time Fourier Transform</a> (STFT). Then subtract noise from the speech signal's STFT:
```
def wave2stft(wavefile):
    y, sr = librosa.load(wavefile)
    if len(y)%2 != 0:
        y = y[:-1]
    stft = librosa.stft(y)
    stft = np.transpose(stft)
    return stft, y, sr

def stft2power(stft_matrix):
    stft = stft_matrix.copy()
    power = np.abs(stft)**2
    return(power)

def reduce_noise(noise_powerspectrum_mean,noise_powerspectrum_variance, speech_powerspectrum_row,speech_stft_row):
    npm = noise_powerspectrum_mean
    npv = noise_powerspectrum_variance
    spr = speech_powerspectrum_row
    stft_r = speech_stft_row.copy()
    for i in range(len(spr)):
        if spr[i] <= npm[i] + npv[i]:
            stft_r[i] = 1e-3
    return stft_r
    
    
noise_STFT, noise_samples, noise_samplingrate = wave2stft("backgroundnoise.wav")
speech_STFT, speech_samples, speech_samplingrate = wave2stft("user_mimic.wav")
noise_powerspectrum = stft2power(noise_STFT)
speech_powerspectrum = stft2power(speech_STFT)

import numpy as np
speech_reducednoise_STFT = np.array([reduce_noise(noise_powerspectrum_mean,noise_powerspectrum_variance,speech_powerspectrum[row],speech_STFT[row]) for row in range(speech_STFT.shape[0])])
```

##### Save the noise reduced STFT/speech signal to wavefile (to play it and make sure it works).
```
def stft2wave(stft,len_origsamp):
    istft = np.transpose(stft.copy())
    samples = librosa.istft(istft,length=len_origsamp)
    return samples
    
speech_reducednoise = stft2wave(speech_reducednoise_STFT,len(speech_samples))
savewave("user_mimic_reducednoise.wav",speech_reducednoise,sr=22050)
```

One way I knew this worked was testing out my game while my vacuuming robot was on, right next to me (I had somehow ignored the little guy - we call him Roby). I was shocked to find basically silent speech recordings! So, needless to say, this game shouldn't be played while vacuuming your apartment.

##### My Mimic with Vacuum in the Background
![Imgur](https://i.imgur.com/dgrsfSP.png)
##### My Mimic Post Noise Reduction:
![Imgur](https://i.imgur.com/XdiJLOD.png)
###### Visualizations created using <a href="https://www.audacityteam.org/">Adacity</a>

Here is an example of how the noise reduction function ideally works:
##### Audio Signal of Dove
![Imgur](https://i.imgur.com/RZoV918.png)
##### User's Mimic Before Noise Reduction
![Imgur](https://i.imgur.com/B79OTih.png)
##### User's Mimic Post Noise Reduction
![Imgur](https://i.imgur.com/juexi3F.png)

You'll notice that the amplitude of the user's recording is still quite a bit higher than the target recording, despite noise reduction. I confronted this problem by developing a function that matched the volume of the user's recording
with that of the target recording:
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
Now a comparison of the target recording and the preprocessed mimic:

##### Audio Signal of Dove
![Imgur](https://i.imgur.com/9JjFU77.png)
##### User's Mimic Post Volume Matching
![Imgur](https://i.imgur.com/jp24Gf8.png)

Much better, right?

A surprise issue I faced was artifacts in collected audio data: in some of the recordings, artifacts were present in the first milliseconds. Similar patterns were found in the course of this research <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5426841/pdf/sensors-17-00917.pdf">project.</a> Since I didn't want these influencing analyses, I decided to remove the first few milliseconds of every recording the game took. 
##### Example of Beginning Recording Artifact
![Imgur](https://i.imgur.com/aeqYaoM.png)

I am currently still working on how to best compare the target sound and the user's mimic. Two different comparison techniques are in the works: 1) Chromaprint and 2) pitch curve analysis. Improvements of these techniques are also in the works. I hope to combine them soon. 

For now, I developed a function that identifies 1) if speech is present and 2) where speech begins (usually a mimicker needs a second to start mimicking). By deleting the lack of speech at the beginning of a recording, that seemed to help both techniques measure similarity better.
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
    
def get_energy(stft_matrix):
    #stft.shape[1] == bandwidths/frequencies
    #stft.shape[0] pertains to the time domain
    rms_list = [np.sqrt(sum(np.abs(stft_matrix[row])**2)/stft_matrix.shape[1]) for row in range(len(stft_matrix))]
    return rms_list    

speech_energy = get_energy(speech_reducednoise_STFT)
voice_start,voice = sound_index(speech_energy,start=True,,start=True,rootmeansquare_mean_noise=noise_energy_mean)
if voice:
    start = voice_start/len(speech_energy)
    start_time = (len(speech_samples)*start)/speech_samplingrate
    speech_reducednoise_STFT = speech_reducednoise_STFT[voice_start:]
    voicestart_samp = stft2wave(speech_reducednoise_STFT,len(speech_samples))
    date = get_date()
    savewave('./processed_recordings/rednoise_speechstart_{}.wav'.format(date),voicestart_samp,sr)
    print('Removed silence from beginning of recording. File saved.')
```
Alrighty. All together this is what the game does:

It records a little bit to collect the user's background noise. Here's what my quiet apartment sounds like recorded by my computer:

##### Collects User's Background Noise
![Imgur](https://i.imgur.com/jxT78f9.png)

Next it plays a sound for the user to mimic. This time it's a lion roaring:

##### Lion Roaring
![Imgur](https://i.imgur.com/rSOUomD.png)

Once that plays, the application records the user mimicking that sound. Here's me roaring.

##### Aislyn Roaring
![Imgur](https://i.imgur.com/QqUBKsf.png)

Aaaaaaaah! That looks horrible. Now the preprocessing steps get to work. 

##### Subtract background noise from user's mimic
![Imgur](https://i.imgur.com/k1bwnAF.png)
##### Remove the beginning silence
![Imgur](https://i.imgur.com/OWkNr4E.png)
##### And match the volume to that of the roaring lion
![Imgur](https://i.imgur.com/ecnm6rB.png)

As you can see, it's still not perfect; there are still artefacts towards the ends of recordings, which I need to take care of (I will, I will). But these steps have helped the game to measure similarity somewhat. 

Take this Lion series as an example. When comparing the acoustic fingerprint similarity of these, the mimic that underwent all of the preprocessing steps got a higher similarity score.

```
Similarity score of the Lion_Original and the Lion Original: 100
Similarity score of the Mimic1 and the Lion Original: 45
Similarity score of the Mimic2 and the Lion Original: 45
Similarity score of the Mimic3 and the Lion Original: 46
Similarity score of the Mimic4 and the Lion Original: 47

```

#### Acoustic Fingerprints:
##### Lion Roaring
![Imgur](https://i.imgur.com/0rSEH33.png)
##### Aislyn Roaring - volume matched, beginning silence removed, and noise reduction
![Aislyn Roaring: noise reduced, beginning silence removed, volume matched](https://i.imgur.com/mPWqSHv.png)
###### I'm not going to bore you with the other fingerprints - they all looked very similar. This one was the one that got the best score.

The fingerprints and their scores were achieved with the following code (original 
<a href="https://yohanes.gultom.me/2018/03/24/simple-music-fingerprinting-using-chromaprint-in-python/">source</a>):

```
import acoustid
import chromaprint

lion_original = 'lion_roar.wav'
lion_mimic = 'aislyn_roar.wav'
lionmimic_post_noisereduction = 'aislyn_roar_noise_reduction.wav'
lionmimic_post_speechstart = 'aislyn_roar_speech_start.wav'
lionmimic_post_volumematch = 'aislyn_roar_volume_matched.wav'

waves2compare = [lion_original,lion_mimic,lionmimic_post_noisereduction,lionmimic_post_speechstart,lionmimic_post_volumematch]
labels = ['Lion_Original','Mimic1','Mimic2','Mimic3','Mimic4']

#collect fingerprints
fingerprints = []
fp_encodings = []
for wave in waves2compare:
    _,fingerprint_encoded = acoustid.fingerprint_file(wave)
    fingerprint, _ = chromaprint.decode_fingerprint(fingerprint_encoded)
    fingerprints.append(fingerprint)
  
import numpy as np
import matplotlib.pyplot as plt
#visualize fingerprints and save to .png
for j in range(len(fingerprints)):
    fig = plt.figure()
    bitmap = np.transpose(np.array([[b == '1' for b in list('{:32b}'.format(i & 0xffffffff))] for i in fingerprints[j]]))
    plt.imshow(bitmap)
    fig.savefig('fingerprint_{}.png'.format(labels[j]))
    
from fuzzywuzzy import fuzz

scores = [fuzz.ratio(fp,fingerprints[0]) for fp in fingerprints] 
 
for i in range(len(scores)):
    print("Similarity score of the {} and the Lion Original: {}".format(labels[i],scores[i]))
```

If we look at how the pitch curve analysis works, the problem will make itself quite apparent. For now I am comparing the area underneath the pitch curves. I knew that this wouldn't be perfect but it was the quickest solution I could come up with. 

Let's first compare the pitch curves of the lion and my mimic:
##### Lion's Pitch Curve
![Imgur](https://i.imgur.com/AkGi9E4.png)
##### Aislyn's Unprocessed Pitch Curve
![Imgur](https://i.imgur.com/qrsty5C.png)
##### Aislyn's Preprocessed Pitch Curve
![Imgur](https://i.imgur.com/DGNUcHZ.png)

Now for comparison, the pitch curves of a cute meowing cat and me sounding ridiculous:
##### Cute Cat's Pitch Curve
![Imgur](https://i.imgur.com/QCdEBRm.png)
##### Ridiculous Aislyn's Pitch Curve (unprocessed)
![Imgur](https://i.imgur.com/kcFeGn4.png)
##### Ridiculous Aislyn's Pitch Curve (preprocessed)
![Imgur](https://i.imgur.com/ikMwOAC.png)

I would say my mimic of the cat was better than that of the lion. Well, according to the pitch curve analysis, it wasn't by a long shot. 

From -1 to 1, I scored a 0.59 on my lion mimic, and only a -0.49 on my cat mimic. 

That's because if the curves aren't perfectly lined up, the area cancels out. So.... ya. I'm working on that. Other methods of similarity such as <a href="https://stackoverflow.com/questions/21647120/how-to-use-the-cross-spectral-density-to-calculate-the-phase-shift-of-two-relate">cross-spectral density</a> or <a href="https://perso.limsi.fr/mareuil/publi/IS110831.pdf">Dynamic Time Warping</a> need to be implemented perhaps instead.

This was a bit long-winded but I hope it was a little enlightening, the turmoil I've gone through in developing what seemed such a simple game. 


### Articles or Resources I found helpful:
* Easy visualization and comparison of Chromaprint <a href="https://yohanes.gultom.me/2018/03/24/simple-music-fingerprinting-using-chromaprint-in-python/">fingerprints</a>.
