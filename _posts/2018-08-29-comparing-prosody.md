---
layout: post
title: "Walking through equations: comparing pitch and prosody of two audio signals"
date: 2018-08-29
---

This post expands on the work of my <a href="https://a-n-rose.github.io/2018/08/24/mimic-master-pitchcurve-vs-fingerprint.html">Mimic Master</a> game, where I compare the signal of an animal's call and that of a human's mimic of that sound. Below I will try different ways of measuring the similarity of the signals' pitch and prosody.

In the game's first version, I collected the pitches of the signals with librosa, using librosa's default values:
```
y,sr = librosa.load(wavefile)
pitches, magnitudes = librosa.piptrack(y)
pitches = np.transpose
```
I then took the mean of the pitches along each row (i.e. along the time axis) and graphed their square-roots. This resulted in a graph that scaled the pitch curves to a similar level and kept from large jumps in pitch to catch too much of our attention. Below is what the graphs look like. 
##### Pitch Curve: Cute Cat Meowing
![Imgur](https://i.imgur.com/Iu5NcpB.png)
##### Pitch Curve: Human Meowing
![Imgur](https://i.imgur.com/f5UnwSb.png)
##### Pitch Curve: Human Cacka-doodle-doing (Rooster Mimic)
![Imgur](https://i.imgur.com/PSO4CPL.png)
###### Cacka-doodle-dooooo

To produce similarity scores, I used the Pearson's correlation coefficient. This was not very successful; even if two pitch curves had similar patterns, if they were not perfectly aligned, their dissimilarity would be overexaggerated. (For the code, see the first version of the <a href="https://a-n-rose.github.io/2018/08/24/mimic-master-pitchcurve-vs-fingerprint.html">Mimic Master</a> game.)

The similarity score of the Cat and Cat Mimic came to:
* -0.6818043886826649

The similarity score of the Cat and Rooster Mimic came to:
* -0.2219641856228962

Clearly the Cat and Cat Mimic should be more similar than a mimic of some random animal.

To explore ways to improve this score, I used this <a href="https://perso.limsi.fr/mareuil/publi/IS110831.pdf">paper</a> as inspiration. The authors compared using a Hermes similarity measure with a Dynamic Time Warped (DTW) algorithm, the former using primarily local measurements (i.e. comparing pitch values of speech signals at 1 millisecond intervals) and the latter using a combinataion of local measurements as well as 'long-term' measurements (i.e. measuring values in the speech signal at windows of 256ms, at 1 millisecond intervals).

Below is how I applied the Hermes similarity measurement:

1) The necessary defintions:
```
def wave2stft(wavefile):
    y, sr = librosa.load(wavefile)
    if len(y)%2 != 0:
        y = y[:-1]
    stft = librosa.stft(y,hop_length=int(0.001*sr))
    stft = np.transpose(stft)
    return stft, y, sr

def stft2power(stft_matrix):
    stft = stft_matrix.copy()
    power = np.abs(stft)**2
    return(power)

def get_pitch2(y,sr):
    pitches,mag = librosa.piptrack(y=y,sr=sr,hop_length=int(0.001*sr))
    return pitches,mag

def match_len(matrix_list):
    matrix_lengths = [len(matrix) for matrix in matrix_list]
    min_index = np.argmin(matrix_lengths)
    newlen = matrix_lengths[min_index]
    matched_matrix_list = []
    for item in matrix_list:
        item = item[:newlen]
        matched_matrix_list.append(item)
    return matched_matrix_list
```

2) Calculate the two signals' pitch and sum of powers (stft = Short-Time Fourier Transform, which is needed for calculating the power)
```
mstft,m,sr = wave2stft(mimic)
mp,mmag = get_pitch2(m,sr)
mpitch = np.transpose(mp)
mpower = stft2power(mstft)

astft,a,sr = wave2stft(animal_sound)
ap,amag = get_pitch2(a,sr)
animal_pitch = np.transpose(ap)
apower = stft2power(astft)

pitch_list1 = match_len([mimic_pitch,animal_pitch])
power_list1 = match_len([mimic_power,animal_power])
sumpower = list(map(sum, power_list1))
```

3) Now apply the Hermes weighted correlation equation:

```
coefficients = []
for i in range(len(sumpower)):
    nom = sum(sumpower[i]*((mimic_pitch[i]-np.mean(mimic_pitch))*(animal_pitch[i]-np.mean(animal_pitch))))
    den = np.sqrt(sum(sumpower[i]*((mimic_pitch[i]-np.mean(mimic_pitch))**2))*sum(sumpower[i]*((animal_pitch[i]-np.mean(animal_pitch))**2)))
    coefficients.append(nom/den)
    
sum(coefficients)
```

Using the Hermes weighted coefficient, here were the results:

The Cat and Cat Mimic
* -0.5827660456045796

The Cat and Rooster Mimic
* -0.49357990451239103

The Rooster Mimic now barely scores to be more similar. An improvement at least in that respect.

For the next step, I tried applying similar calculations but not only at the local level of 1 millisecond intervals but rather at the long-term level, with 256ms windows at each millisecond interval (similar to the DTW algorithm applied in the paper). 

```
def long_term_info(y,sr):
    stft = librosa.stft(y,hop_length=int(0.001*sr),n_fft=int(0.256*sr))
    stft = np.transpose(stft)
    power = np.abs(stft)**2
    pitch, mag = librosa.piptrack(y=y,sr=sr,hop_length=int(0.001*sr),n_fft=int(0.256*sr))
    pitch = np.transpose(pitch)
    return stft,power,pitch
 
m, sr = librosa.load(mimic_wavefile)
a, sr = librosa.load(animal_wavefile)

mstft_long,mpower_long,mpitch_long = long_term_info(m,sr)
astft_long,apower_long,apitch_long = long_term_info(a,sr)
mpower_long,apower_long = match_len(mpower_long,apower_long)
mpitch_long,apitch_long = match_len(mpitch_long,apitch_long)
sumpower_long = mpower_long+apower_long

coefficients_longterm = []
for i in range(len(sumpower)):
    nom = sum(sumpower_long[i]*((mpitch_long[i]-np.mean(mpitch_long))*(apitch_long[i]-np.mean(apitch_long))))
    den = np.sqrt(sum(sumpower_long[i]*((mpitch_long[i]-np.mean(mpitch_long))**2))*sum(sumpower_long[i]*((apitch_long[i]-np.mean(apitch_long))**2)))
    coefficients_longterm.append(nom/den)
    
sum(coefficients_longterm)
```

This resulted in the scores:

Cat vs Cat Mimic
* 0.2313280069785804

Cat vs Rooster Mimic
* -0.1367108903732711

Wow! Including the window of 256ms made already a huge difference! One problem: the computation time is too long for a silly mimic game. 

A comparison of calculation times (of pitch and STFT) and scores with varying millisecond intervals and window lengths (local vs long-term i.e. 0ms vs 256ms):




### Calculation times and scores with Librosa's default Values:
Pitch: 
* 0.09511232376098633 seconds

STFT:
* Not computed

Similarity score for the matched mimic: -0.6818043886826649

Similarity score for the random mimic: -0.2219641856228962

### Calculation times and scores with 1 millisecond intervals:
Pitch
* 0.6700444221496582 seconds

STFT:
* 0.2541377544403076 seconds

Similarity score for the matched mimic: -0.5827660456045796

Similarity score for the random mimic: -0.49357990451239103

### Calculation times and scores with 1 millisecond intervals and 256 ms windows
Pitch:
* 2.5810253620147705 seconds

STFT:
* 1.2656512260437012 seconds

Similarity score for the matched mimic: 0.2313280069785804

Similarity score for the random mimic: -0.1367108903732711

### Calculation times and scores with 10 millisecond intervals:

Pitch: 
* 0.0620427131652832 seconds

STFT:
* 0.0929727554321289 seconds

Similarity score for the matched mimic: -0.5672101641613938

Similarity score for the random mimic: -0.43661382558278505

### Calculation times and scores with 10 millisecond intervals and 256 ms windows:

Pitch: 
* 0.20823955535888672 seconds

STFT:
* 0.1283879280090332 seconds


Similarity score for the matched mimic over longer intervals: 0.1726895461780784

Similarity score for the random mimic over longer intervals: -0.11110679757542691


Comparing the calcuation times and the scores, it looks like using the 10 millisecond intervals with 256 millisecond window lengths is the way to go. Before we decide to integrate that one, I would like to see how these settings compare with other sounds. Due to long processing times of the 1 second interval calculations, I will not include those. 


### Deerkill Call

#### Original settings (Librosa Defaults)

Similarity score for the matched mimic: -0.09117716135298223
Similarity score for the random mimic: -0.13732110604409462


#### 10 ms intervals, no window:
Similarity score for the matched mimic: 0.42417864186059656
Similarity score for the random mimic: 0.08138676704482051


#### 10 ms intervals, 256 ms windows

Similarity score for the matched mimic: 0.3183987199215531
Similarity score for the random mimic: 0.34028664905198897

For this mimic, it looks like the 10 ms intervals without windows worked better than the one with a 256 ms window. The authors of the paper used a combination of local and long-term values, which might be necessary in this game as well. Let's see how it does with giving this sound another try

### Killdeer Call (Attempt 2)

#### Original settings (Librosa Defaults)

Similarity score for the matched mimic: -0.27329152650559585
Similarity score for the random mimic:  -0.13732110604409462


#### 10 ms intervals, no window:
Similarity score for the matched mimic: 0.2673424830174177
Similarity score for the random mimic: 0.08138676704482051


#### 10 ms intervals, 256 ms windows

Similarity score for the matched mimic: 0.6366381765045449
Similarity score for the random mimic: 0.34028664905198897

Hmmmm... now the one with the 256 ms windows did better. Let's keep comparing.


### Kitten Meow (different from the Cat above)
Similarity score for the matched mimic: 
Similarity score for the random mimic: 
Similarity score for the matched mimic over longer intervals: 
Similarity score for the random mimic over longer intervals: 


#### Original settings (Librosa Defaults)

Similarity score for the matched mimic: 0.8284534278816392
Similarity score for the random mimic: 0.08433401056024156


#### 10 ms intervals, no window:
Similarity score for the matched mimic: 0.13376621453344462
Similarity score for the random mimic: 0.2754590323085617


#### 10 ms intervals, 256 ms windows

Similarity score for the matched mimic: 0.016711484170983434
Similarity score for the random mimic: -0.0664781439969252


Uuuuuuuuuugh. In this sound-mimic comparison, a lot of the signal was 'silent'. That means I've got to improve how silence is processed, which sounds easier than it is.

### Lion

#### Original settings (Librosa Defaults)

Similarity score for the matched mimic: 0.7050785658886863
Similarity score for the random mimic: 0.14895963871760315


#### 10 ms intervals, no window:
Similarity score for the matched mimic: 0.17497106429155274
Similarity score for the random mimic: 0.024416334848876273


#### 10 ms intervals, 256 ms windows

Similarity score for the matched mimic: 0.4595252037839041
Similarity score for the random mimic: -0.35400277458327795


All in all pretty good here.

### Pig

Similarity of pitch curve between the original sound and pig_mimic: 
Similarity of pitch curve between the original sound and pig_random: 
Similarity score for the matched mimic: 
Similarity score for the random mimic: 
Similarity score for the matched mimic over longer intervals: 
Similarity score for the random mimic over longer intervals: 

#### Original settings (Librosa Defaults)

Similarity score for the matched mimic: 0.21847745592292092
Similarity score for the random mimic: 0.110099939161723


#### 10 ms intervals, no window:
Similarity score for the matched mimic: 0.17149208332492877
Similarity score for the random mimic: 0.519814545647899


#### 10 ms intervals, 256 ms windows

Similarity score for the matched mimic: -0.08139453758826998
Similarity score for the random mimic: 0.2660859135026845


I have to admit, this was a tough mimic.


### Horse


#### Original settings (Librosa Defaults)

Similarity score for the matched mimic: -0.401542933730961
Similarity score for the random mimic: 0.23835958982283567


#### 10 ms intervals, no window:
Similarity score for the matched mimic: -0.09761662974599129
Similarity score for the random mimic: -0.0479235452071661


#### 10 ms intervals, 256 ms windows

Similarity score for the matched mimic: -0.13517197084986413
Similarity score for the random mimic: -0.36793423365350353

I'm guessing sounds with stronger pitches are easier to compare! I wonder how these would do when comparing acoustic fingerprints....
