---
layout: post
title: "Mimic Master - First Attempt: Learning the wrong way to compare speech signals"
date: 2018-08-24
---

Imagine yourself bored on your couch, wishing you were hiking in the mountains, hearing birds sing and returning their calls. In your fantasy you respond perfectly, but wonder how well you actually do that in reality. Have I got the solution for you! You can test your mimics with this silly little mimic game.

Here's how it works. 


First it records your own environment noise (to subtract it later from your amazing mimics). Mine looks like this:

#### Collects User's Background Noise
![Imgur](https://i.imgur.com/0PfFvoF.png)

Next it plays a sound. This time it's a roaring lion:
#### Lion Roaring
![Imgur](https://i.imgur.com/rSOUomD.png?1)

And now the user shows off their skills:

#### Me Roaring
![Imgur](https://i.imgur.com/QqUBKsf.png?1)

Aaaaaaaah! That looks horrible. Let's neaten this up a bit. 

![Imgur](https://i.imgur.com/ecnm6rB.png?1)

Few. Much better. 

How well did I do? 
I got two scores: 

1) 47

2) 0.59

(I'll get into the scoring later on)

Next try. 

Awe cute... it's a meowing cat
![Imgur](https://i.imgur.com/bxBYaAG.png?1)

A meowing bored person:
![Imgur](https://i.imgur.com/HyWid7b.png?1)

A little touched up:
![Imgur](https://i.imgur.com/higYIak.png?1)

And the score(s) are....

1)  50

2) -0.6818043886826649

Alright... I improved according to the first score but performed vastly worse according to the second score.

What happens if a user makes random sounds? Like mimicking a rooster instead of a lion?

#### Lion:
![Imgur](https://i.imgur.com/rSOUomD.png?1)
#### Rooster Mimic:
![Imgur](https://i.imgur.com/CkIKDBr.png?1)
###### Cacka-doodle-dooooo

Scores:

1) 48

2) 0.15282471566864583

Hmmmmmmmmmmmm... This isn't encouraging. Well, to see where developing this game came of any use, check out how I applied my noise cancelling function to increase accuracy of a deep learning classifier <a href="https://a-n-rose.github.io/2018/08/23/noise-how-i-love-you.html">here.</a> To see the hurdles I went through in developing this (so far unsuccesful) game, read on!

### The Development Process

This has been a blast and adventure to build. I have come across several problems, ranging from differencs in microphone quality to detecting speech amidst noise, let alone comparing pitch or prosody of two audio signals. 

Any app that records a user remotely has to deal with varying microphone quality and background noise levels. My first challenge was finding a simple way to cancel out whatever background noise a user might have due to those variables. 

I took on the task to build by own version of <a href="https://www.audacityteam.org/">Audacity's</a> noise reduction technique; I would say I was successful as background noise was removed from the user's mimics, which improved the analysis of their speech. (Audacity's software includes smoothing and other more complicated techniques, which I didn't do, but my functions worked just fine for the task at hand.) In the game, I started out by testing the user's mic by recording 5 seconds of their background noise. I calculated the spectral power in that recording and subtracted it from all of the user's subsequent mimics. 

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

#### My Mimic with Vacuum in the Background
![Imgur](https://i.imgur.com/dgrsfSP.png?1)
#### My Mimic Post Noise Reduction:
![Imgur](https://i.imgur.com/XdiJLOD.png?1)


Here is an example of how the noise reduction function works without a vacuum running in the background:
#### Audio Signal of Dove (animal sound to mimic)
![Imgur](https://i.imgur.com/9JjFU77.png?1)
#### User's Mimic Before Noise Reduction
![Imgur](https://i.imgur.com/B79OTih.png?1)
#### User's Mimic Post Noise Reduction
![Imgur](https://i.imgur.com/juexi3F.png?1)

You'll notice that the amplitude of the user's recording is still quite a bit higher than the target recording, despite noise reduction. I confronted this problem by developing a function that matched the volume of the user's recording with that of the target recording:
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
Now a comparison of the target recording and the cleaned-up mimic:

#### Audio Signal of Dove 
![Imgur](https://i.imgur.com/9JjFU77.png?1)
#### User's Mimic Post Volume Matching
![Imgur](https://i.imgur.com/jp24Gf8.png?1)

Much better, right?

Similar to the scores above, I wasn't seeing my rating system produce any meaningful scores (I'll get to that in a bit). So I decided to cut out the starting silence of the user's mimics. Perhaps that would help produce more accurate scores.

Below is a function that identifies 1) if speech is present and 2) where speech begins (usually a mimicker needs a second to start mimicking).
```
def sound_index(rootmeansquare_speech, start = True, rootmeansquare_mean_noise = None):
    # options for using background noise or not
    if rootmeansquare_mean_noise == None:
        rootmeansquare_mean_noise = 1
    
    # option for finding the beginning or end of speech
    # if start == True, start calculating from 0 --> end 
    # if start == False, start calcuating from end --> 0
    if start == True:
        side = 1
        beg = 0
        end = len(rootmeansquare_speech)
    else:
        side = -1
        beg = len(rootmeansquare_speech)-1
        end = -1
    
    # now see if/when power in mimic exceeds that of the noise signal
    # note: 'row' represents a step in time
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
    # stft.shape[1] == bandwidths/frequencies
    # stft.shape[0] pertains to the time domain
    rms_list = [np.sqrt(sum(np.abs(stft_matrix[row])**2)/stft_matrix.shape[1]) for row in range(len(stft_matrix))]
    return rms_list    

speech_energy = get_energy(speech_reducednoise_STFT)
voice_start,voice = sound_index(speech_energy,start=True,rootmeansquare_mean_noise=noise_energy_mean)
if voice:
    # find at which percentile the speech starts in energy spectrum:
    start = voice_start/len(speech_energy)
    # use that percentile to find the start in time domain:
    start_time = (len(speech_samples)*start)/speech_samplingrate
    
    # get stft starting from where speech starts
    speech_reducednoise_STFT = speech_reducednoise_STFT[voice_start:]
    
    # perform iSTFT (stft2wave) to get sample values --> save to wavefile
    voicestart_samp = stft2wave(speech_reducednoise_STFT,len(speech_samples))
    date = get_date()
    savewave('./processed_recordings/rednoise_speechstart_{}.wav'.format(date),voicestart_samp,sr)
    print('Removed silence from beginning of recording. File saved.')
```

Additionally I noticed artefacts (little clicks) in the speech recordings, particularly in the beginning. Therefore I removed the beginng few milliseconds to remove them. (Unfortunately, other artefacts, for example at the ending of recordings, that I still need to account for.)
#### Artefacts I removed during processing
![Imgur](https://i.imgur.com/aeqYaoM.png?1)
#### Artefacts I still need to account for 
![Imgur](https://i.imgur.com/CkIKDBr.png?1)
###### (see towards the end of the recording)

### The Scores

### First Score
The first score was generated by a comparison of acoustic fingerprints. (Original <a href="https://yohanes.gultom.me/2018/03/24/simple-music-fingerprinting-using-chromaprint-in-python/">source</a>):

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

And here's what the fingerprints looked like:

#### Lion 
![Imgur](https://i.imgur.com/0rSEH33.png)
#### Me mimicking the lion
![Aislyn Roaring: noise reduced, beginning silence removed, volume matched](https://i.imgur.com/mPWqSHv.png)
###### Note: the mimic fingerprint is a bit wider due to the longer length. The mimics were 1 second longer than the original sound to account for user response delay.

If I'm honest here, I was mainly just testing Chromaprint out to see if it worked in this setting, without too much additional work. I don't know so much about the workings behind the scenes, so clearly I have some more to explore and read about when creating and comparing acoustic fingerprints.

### Second Score

Another quick solution I tried out was comparing the pitch curves using the "Pearson product-moment correlation coefficients". Basically I compared the areas under the pitch curves of the target sound and the mimic. I knew this wouldn't be perfect but I thought that it might produce somewhat reliable scoring, especially if I remove the beginning silences of the user's mimics, to ensure the pitch curves lined up somewhat.

Well... yeah. Any sort of curve mismatch distorted how poor the mimic was. Therefore, I am now exploring more complicated and interesting ways of comparing similarity such as <a href="https://stackoverflow.com/questions/21647120/how-to-use-the-cross-spectral-density-to-calculate-the-phase-shift-of-two-relate">Cross-Spectral Density</a> and <a href="https://perso.limsi.fr/mareuil/publi/IS110831.pdf">Dynamic Time Warping</a>. 

Note: For the pitch-curve generation, I collected pitch values with librosa's piptrack module, then took their means (along the time-domain), as well as the square roots of their means. This made large jumps in the pitch curve less influential of the general pattern of the sound. Below are some graphs and code, for those who are interested.

#### Pitch Curve: Lion Roar
![Imgur](https://i.imgur.com/DraESrw.png)
#### Pitch Curve: Lion Mimic (unprocessed) 
![Imgur](https://i.imgur.com/x8aOkfy.png)
#### Pitch Curve: Lion Mimic (preprocessed) 
![Imgur](https://i.imgur.com/I9sA6IP.png)

Now for comparison, the pitch curves of a cute meowing cat and me sounding ridiculous:
#### Pitch Curve: Cute Cat
![Imgur](https://i.imgur.com/Iu5NcpB.png)
#### Pitch Curve: Cat Mimic (unprocessed)
![Imgur](https://i.imgur.com/pnAzH5U.png)
#### Pitch Curve: Cat Mimic (preprocessed)
![Imgur](https://i.imgur.com/0dMbSXG.png)

```
import librosa
import numpy as np
import matplotlib.pyplot as plt

def get_pitch_mean(matrix_pitches):
    p = matrix_pitches.copy()
    p_mean = [np.mean(p[:,time_unit]) for time_unit in range(p.shape[1])]
    p_mean = np.transpose(p_mean)
    #remove beginning artifacts:
    pmean = p_mean[int(len(p_mean)*0.07):]
    return pmean
              
def pitch_sqrt(pitch_mean):
    psqrt = np.sqrt(pitch_mean)
    return psqrt

def get_pitch(wavefile):
    y, sr = librosa.load(wavefile)
    if len(y)%2 != 0:
        y = y[:-1]
    pitches,mag = librosa.piptrack(y=y,sr=sr)
    return pitches,mag

animal_original = 'cat_meow.wav'
animal_mimic = 'aislyn_meow.wav'
animalmimic_post_noisereduction = 'aislyn_roar_noise_reduction.wav'
animalmimic_post_speechstart = 'aislyn_roar_speech_start.wav'
animalmimic_post_volumematch = 'aislyn_roar_volume_matched.wav'

animal_type = 'cat'
labels = ['original_sound_{}'.format(animal_type),'mimic1_{}'.format(animal_type),'mimic2_{}'.format(animal_type),'mimic3_{}'.format(animal_type),'mimic4_{}'.format(animal_type)]

waves_list = [animal_original,animal_mimic,animalmimic_post_noisereduction,animalmimic_post_speechstart,animalmimic_post_volumematch]

pitches = [get_pitch(wave) for wave in waves_list]
pitch_means = [get_pitch_mean(pitch[0]) for pitch in pitches]
pitch_means_sqrts = [pitch_sqrt(mean) for mean in pitch_means]

for j in range(len(pitch_means_sqrts)):
    fig = plt.figure()
    plt.plot(pitch_means_sqrts[j])
    fig.savefig('pitchcurve_{}.png'.format(labels[j]))

def compare_sim(pitch_mean1, pitch_mean2):
    pm1 = pitch_mean1.copy()
    pm2 = pitch_mean2.copy()
    if len(pm1) != len(pm2):
        index_min = np.argmin([len(pm1),len(pm2)])
        if index_min > 0:
            pm1 = pm1[:len(pm2)]
        else:
            pm2 = pm2[:len(pm1)]
    corrmatrix = np.corrcoef(pm1,pm2)
    return(corrmatrix)

pitch_sim = [compare_sim(sqrt,pitch_means_sqrts[0]) for sqrt in pitch_means_sqrts]
for i in range(len(pitch_sim)):
    print("Similarity of pitch curve between the original sound and {}: {}".format(labels[i],pitch_sim[i][0][1]))

```

In sum, I'm happy with results so far, largely because I could use functions I wrote for this game for other applications. This was the main reason why I developed this game: to learn how to process, analyze, and manipulate speech. I still have a ways to go, however, to get this to a point of accurately rating mimics. 


~ Speech signal visualizations created using <a href="https://www.audacityteam.org/">Audacity</a> 

