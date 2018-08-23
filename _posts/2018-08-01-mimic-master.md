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
![Imgur](https://i.imgur.com/RZoV918.png)
##### User's Mimic Post Volume Matching
![Imgur](https://i.imgur.com/jp24Gf8.png)

Much better, right?

A surprise issue I faced was artifacts at the start of recordings: in some of the recordings, artifacts were present in the first milliseconds. Similar patterns were found in the course of this research <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5426841/pdf/sensors-17-00917.pdf">project.</a> Since I didn't want these influencing analyses, I decided to remove the first few milliseconds of every recording the game took. 
##### Example of Beginning Recording Artifact
![Imgur](https://i.imgur.com/aeqYaoM.png)

I am currently still working on how to best compare the target sound and the user's mimic. At first I tried using Chromaprint and generated a score based on how similar that software calculated the target and mimic to be. The results were disappointing though: chance. I could have screamed into the microphone when the target was a whispering panda, and then expertly mimicked a chimpanzee call, our closest animal relative, and ended up with comparable scores. 

Perhaps Chromaprint will work better as I improve how I preprocessing the audiofiles, or perhaps I missed something when I implemented it, but for now I've turned to a much more basic approach: Pearson correlation coefficient, i.e. measuring the area beneath pitch curves. This is far from perfect, though. First, I had to remove the beginning silence from both the target sound and the user's recording to get the pitch curves to start at the same time. But if the user's timing is just slightly off throughout the mimic, that difference in pitch curve will greatly exaggerate the imperfection of the user's mimic. Therefore, other methods of similarity such as <a href="https://stackoverflow.com/questions/21647120/how-to-use-the-cross-spectral-density-to-calculate-the-phase-shift-of-two-relate">cross-spectral density</a> or <a href="https://perso.limsi.fr/mareuil/publi/IS110831.pdf">Dynamic Time Warping</a> need to be implemented.

Another hurdle I'm trying to jump over is how to balance strict enough measures to ensure that if the user says nothing, that is clearly identified by the game, but also able to identify sublties in the user's recording that might get lost if the measures are too rigid. I'm hoping as I improve the comparison methods this will get better as a result. 

For the target sounds, I collected recordings from <a href="https://freesound.org/">freesound</a>, mainly animals 'cause those are the most fun. I am excited to eventually include different kinds of sounds, like famous speakers and non-living sounds. I'm sure many more fun issues will appear with those!




