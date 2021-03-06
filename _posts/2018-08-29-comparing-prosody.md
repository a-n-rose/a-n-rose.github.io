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

To explore ways to improve this score, I used this <a href="https://perso.limsi.fr/mareuil/publi/IS110831.pdf">paper (link to pdf)</a> as inspiration. The authors compared using a Hermes similarity measure with a Dynamic Time Warped (DTW) algorithm. The former uses primarily local measurements (i.e. comparing pitch values of speech signals at 1 millisecond intervals) and the latter uses a combinataion of local measurements as well as 'long-term' measurements (i.e. measuring values in the speech signal at windows of 256ms, at 1 millisecond intervals).

Here is the Hermes equation as applied in the paper:

<a href="https://www.codecogs.com/eqnedit.php?latex=r_{f_1&space;f_2}&space;=&space;\frac{\sum_i&space;w(i)(f_1(i)-m_1)(f_2(i)-m_2)}{\sqrt&space;(\sum_i&space;w(i)(f_1(i)-m_1)^{2}&space;\sum_i&space;w(i)(f_2(i)-m_2)^{2})}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?r_{f_1&space;f_2}&space;=&space;\frac{\sum_i&space;w(i)(f_1(i)-m_1)(f_2(i)-m_2)}{\sqrt&space;(\sum_i&space;w(i)(f_1(i)-m_1)^{2}&space;\sum_i&space;w(i)(f_2(i)-m_2)^{2})}" title="r_{f_1 f_2} = \frac{\sum_i w(i)(f_1(i)-m_1)(f_2(i)-m_2)}{\sqrt (\sum_i w(i)(f_1(i)-m_1)^{2} \sum_i w(i)(f_2(i)-m_2)^{2})}" /></a>

###### f1 and f2 are the fundamental frequencies (pitch) of the target sound and mimic; m1 and m2 represent their means, respectively; w(i) represents the sum of their powers. 

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

def get_pitch(y,sr):
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
# 'm' stands for mimic
mstft,m,sr = wave2stft(mimic)
mp,mmag = get_pitch(m,sr)
mpitch = np.transpose(mp)
mpower = stft2power(mstft)

# 'a' stands for animal sound
astft,a,sr = wave2stft(animal_sound)
ap,amag = get_pitch(a,sr)
animal_pitch = np.transpose(ap)
apower = stft2power(astft)

# collect the pitches of each sound
pitch_list1 = match_len([mimic_pitch,animal_pitch])
# collect the powers of each sound
power_list1 = match_len([mimic_power,animal_power])

# Hermes weighted correlation equation uses the 
# sum of power of the two signals as a weight
sumpower = list(map(sum, power_list1))
```

3) Now apply the Hermes weighted correlation equation:

Sorry for the long list comprehension...

```
coefficients = []
for i in range(len(sumpower)):
    numerator = sum(sumpower[i]*((mimic_pitch[i]-np.mean(mimic_pitch))*(animal_pitch[i]-np.mean(animal_pitch))))
    denominator = np.sqrt(sum(sumpower[i]*((mimic_pitch[i]-np.mean(mimic_pitch))**2))*sum(sumpower[i]*((animal_pitch[i]-np.mean(animal_pitch))**2)))
    coefficients.append(numerator/denominator)
    
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


#### Calculation times (sec) and similarity scores across pitch and stft measurement settings. Wavefile processed: Cat Meow

| Measurement Setting | Time: Pitch  | Time: STFT | Score for MM | Score for RM |
|----------------:|:-------------------:|:----------------:|:-------------------:|:----------------|
|Default setting|0.095|Not computed|-0.68|-0.22|
|1 ms intervals|0.67|0.25|-0.58|-0.49|
|1 ms intervals 256 ms windows|2.58|1.27|0.23|-0.14|
|10 ms intervals|0.06|0.09|-0.57|-0.44|
|10 ms intervals 256 windows|0.21|0.13|0.17|-0.11|

##### MM = Matched Mimic; RM = Random Mimic.


Comparing the calcuation times and similarity scores, it looks like using the 10 millisecond intervals with 256 millisecond window lengths is the way to go. Before we decide to integrate that one, I would like to see how these settings compare with other sounds. Due to long processing times of the 1 second interval calculations, I will not include those. 

#### Similarity Scores across Measurement Settings:

| Sound Mimicked | Default setting | 10 ms intervals | 10 ms intervals 256 windows |
|---------------|:----------------:|:-------------------:|:----------------|
| Killdeer Call | MM: -0.27 RM: -0.14 | MM: 0.27 RM: 0.08 | MM: 0.64 RM: 0.34 |
| Kitten Meow | MM: 0.83 RM: 0.08 | MM: 0.13 RM: 0.28 | MM: 0.017 RM: -0.07 |
| Lion | MM: 0.71 RM: 0.15 | MM: 0.17 RM: 0.02 | MM: 0.46 RM: -0.35 |
| Pig | MM: 0.27 RM: 0.11 | MM: 0.17 RM: 0.52 | MM: -0.08 RM: 0.27 |
| Horse | MM: -0.40 RM: 0.24 | MM: -0.10 RM: -0.05 | MM: -0.14 RM: -0.37 |

##### Matched Mimic (MM) vs Random Mimic (RM) scores of similarity. The MM score should be significantly higher than the RM score

While the scores still aren't perfect, the 10ms setting with 256ms windows offers the most consistency in scoring the mimic more similar to the target sound than the random mimic to the target sound. With that I will integrate this new scoring system in my game and see how it goes!


### Links I found useful

* Writing <a href="https://www.codecogs.com/latex/eqneditor.php">math equations</a> in GitHub friendly markdown
