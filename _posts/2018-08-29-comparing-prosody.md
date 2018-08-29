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
I then took the mean of the pitches along each row (i.e. along the time axis) and graphed their square-roots. This resulted in a graph that scaled the pitch curves to a similar level and kept from large jumps in pitch to catch too much of our attention. Below is what the graphs look like. First the target sound, then the mimic of that sound, and for comparison, an unrelated mimic (i.e. mimic of a rooster).
##### Pitch Curve: Cute Cat Meowing
![Imgur](https://i.imgur.com/QCdEBRm.png)
##### Pitch Curve: Human Meowing
![Imgur](https://i.imgur.com/ikMwOAC.png)
##### Pitch Curve: Human Cacka-doodle-doing (Rooster Mimic)
![Imgur](https://i.imgur.com/Qmx1b45.png)
###### Cacka-doodle-dooooo

To produce similarity scores, I used the Pearson's correlation coefficient. This was not very successful; even if two pitch curves had similar patterns, if they were not perfectly aligned, their dissimilarity would be overexaggerated. (For the code, see the first version of the <a href="https://a-n-rose.github.io/2018/08/24/mimic-master-pitchcurve-vs-fingerprint.html">Mimic Master</a> game.)

The similarity score of the Cat and Cat Mimic came to:
-0.6818043886826649
The similarity score of the Cat and Rooster Mimic came to:
-0.14939653912944015

Clearly the Cat and Cat Mimic should be more similar than a mimic of some random animal.

To explore ways to improve this score, I used this <a href="https://perso.limsi.fr/mareuil/publi/IS110831.pdf">paper</a> as inspiration. The authors compared using a Hermes similarity measure with a Dynamic Time Warped (DTW) algorithm, the former using primarily local measurements (i.e. comparing pitch values of speech signals at 1 millisecond intervals) and the latter using a combinataion of local measurements as well as 'long-term' measurements (e.g. measuring values in the speech signal at windows of 256ms, at 1 millisecond intervals).

Below is how I applied the Hermes similarity measurement:
* Calculate the sum of power of the two signals (sumpower)
* For each of the signals, calculate the pitch at every millisecond as well as pitch mean of the entire signal
```
coefficients = []
for i in range(len(sumpower)):
    nom = sum(sumpower[i]*((mimic_pitch[i]-np.mean(mimic_pitch))*(animal_pitch[i]-np.mean(animal_pitch))))
    den = np.sqrt(sum(sumpower[i]*((mimic_pitch[i]-np.mean(mimic_pitch))**2))*sum(sumpower[i]*((animal_pitch[i]-np.mean(animal_pitch))**2)))
    print(nom/den)
    coefficients.append(nom/den)
    
sum(coefficients)
```

Using the Hermes weighted coefficient, here were the results:
The Cat and Cat Mimic
-0.7441400814665404
The Cat and Rooster Mimic
0.1590280449928146

As with the scores from the first version of the game, the random mimic scored more similar than the cat mimic.

I did not yet implement the author's DTW alorithm immediately; I first applied their processing window of 256ms at every 1 ms interval. Here is an example of that code:
```
def long_term_info(y,sr):
    stft = librosa.stft(y,hop_length=int(0.1*sr),n_fft=int(0.256*sr))
    stft = np.transpose(stft)
    power = np.abs(stft)**2
    pitch, mag = librosa.piptrack(y=y,sr=sr,hop_length=int(0.1*sr),n_fft=int(0.256*sr))
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
    print(nom/den)
    coefficients_longterm.append(nom/den)
    
sum(coefficients_longterm)
```

This resulted in the scores:
Cat vs Cat Mimic
-0.1844086273046544
Cat vs Rooster Mimic
0.15232822492231926

The Rooster Mimic still scores more similar, but at least the difference between the scores has decreased from 0.903168126459355 to 0.3367368522269737.

Still room for improvement. Up next is to implement the full Dynamic Time Warped Algorithm. For this I need to measure pitch slope, kurtosis, skewness, standard deviation, mean, and 1st and 9th deciles in each of the 256 windows.






