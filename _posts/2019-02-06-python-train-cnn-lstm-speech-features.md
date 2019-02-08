---
layout: post
title: "Speech and Deep Learning: What does speech look like and how can we prep it for taining?"
date: 2019-02-06
---

This post complements a <a href="https://github.com/a-n-rose/Build-CNN-or-LSTM-or-CNNLSTM-with-speech-features">workshop repo</a> where you can explore speech feature extraction and deep neural network training. To better understand some of the speech features used in machine and deep learning then read on!

Speech and machine learning. Why is it relevant? I assume you're familiar with Siri and Alexa, and (depending on your language/accent) they do what you say, from turning on the lights, playing a song, to taking notes for you. This speech recognition technology <a href="https://uwaterloo.ca/global-impact/how-machine-learning-helps-siri-and-alexa-understand-you">already uses deep learning</a>, but let's say we want to improve it's <a href="https://www.isca-speech.org/archive/Interspeech_2018/pdfs/1864.pdf">understanding of accented speech</a> or to expand it to also identify other aspects of our speech. This can be useful (and is being explored) for health related apps, such as the tracking of <a href="http://www.infomus.org/Events/proceedings/ACII2015/papers/Main_Conference/M4_Doctoral_Consortium/D01_Recognition/ACII2015_submission_195.pdf">depression</a> or <a href="https://ac.els-cdn.com/S2352872915000160/1-s2.0-S2352872915000160-main.pdf?_tid=d04d9cc3-f992-4820-af98-f083c847c322&acdnat=1549408007_fe358db560e7df5618e8d68875824413">identifying the onset of Alzheimer's</a>.

So how can we train a model to identify characteristics in our speech? We need to present whatever machine learning algorithm(s) we want to use speech in a way it learns best from. 

## Speech Signal

### Waveform 

Let's see what information is available in the raw waveform of speech:


![Imgur](https://i.imgur.com/sSYOMbq.png)
#### wave pulled from <a href="https://ai.googleblog.com/2017/08/launching-speech-commands-dataset.html">Speech Commands Dataset</a>


What information can we gather from this picture with our naked eye? 

We can see how much energy is in the signal, and when different speech sounds are made. The beginning fuzzy part looks like it matches to the sound 'sh'. But can we answer these questions: Which exact sounds is this person saying? Could I tell 'sh' from 'z' if I didn't know already that the word is 'Sheila'? Is this person male, female, a child or an adult?

Let's zoom in to see what the actual waves look like; maybe that will tell us something.

![Imgur](https://i.imgur.com/zWqjgkz.png)

Was this what you expected? These waves are pretty squiggly. Compare these waves with those below. 

![Imgur](https://i.imgur.com/m7HmVna.png)

What's the difference? Simply put, the sine wave has just one frequency; the speech sound wave has multiple frequencies, packed together. Here are two more sine waves, at differing frequencies:

![Imgur](https://i.imgur.com/bWTarkl.png)

![Imgur](https://i.imgur.com/GwXFRQd.png)

And here are all three waves, packed together:

![Imgur](https://i.imgur.com/bo6PMnL.png)


For us humans, and most standard machine learning algorithms, it is very hard to tell which frequencies are in there. 

This is where the Fourier Transform comes in!! 

### Fourier Transform: Switch Domains

For a really great explanation of the Fourier Transform, as well as some enjoyable visuals, watch this <a href="https://www.youtube.com/watch?v=spUNpyF58BY">video</a>.

Long story short, by applying this equation - equipped with several spiraling sinewaves, all with different frequencies - to the sound wave, it basically unpacks the frequencies from the sound wave. This can tell us the frequencies in the speech.

Why would frequencies be helpful? Think of how you tell if a speaker is a man, woman, or child. The most obvious way is hearing how high or low the speech is. That's one thing frequencies can tell us but they can potentially reveal additional characteristics, such as health.

### "Dominant Frequencies" of child, adult female, and adult male, all saying the vowel 'a':
![Imgur](https://i.imgur.com/71C4E9l.png)
#### Child: frequency stays around 1200 Hz (wavefile collected from the LANNA database)
![Imgur](https://i.imgur.com/stVda2u.png)
#### Adult female: frequency stays around 1000 Hz (wavefile collected from Saarbrücker Voice Database)
![Imgur](https://i.imgur.com/p6QvgTj.png)
#### Adult male: frequency stays around 700 Hz (wavefile collected from the Saarbrücker Voice Database)

Note: my background is more focused on clinical speech, so I will continue with the standards I've seen for handling speech; I can't (yet) speak for the standards for music or other realms of digital signal processing.

Anyways, while the Fourier Transform is amazing, it expects the signal to be unchanging. Think of the humming comming from your computer, or refrigerator. (It's my microwave's ringing that drives me bonkers.) Those signals stay relatively the same over time.

Speech is vastly different. The signal changes in the realm of milliseconds! These changes reflect the different speech sounds, or phonemes, we make. So, how do we find out the frequencies within such a varying signal? 

### STFT

Apply the Fourier Transform in tiny little windows, specifically, the <a href="https://ccrma.stanford.edu/~jos/sasp/Short_Time_Fourier_Transform.html">short-time fourier transform</a> (STFT)

Most research papers I've come across apply the STFT in windows of 25 milliseconds, meaning they 'transform' the signal at only 25 ms at a time. In order to catch variations in frequency that might happen within that 25 milliseconds, the STFT is calculated in shifts of 10 milliseconds, going down the signal until all of it has been 'transformed'. You'll find functions using this window size and window shift in this <a href="https://github.com/a-n-rose/Build-CNN-or-LSTM-or-CNNLSTM-with-speech-features/blob/master/feature_extraction_scripts/feature_extraction_functions.py">script</a>. 

Knowing this, let's compare the raw waveform of 'sheila' and it's transformed state:

![Imgur](https://i.imgur.com/sSYOMbq.png)


![Imgur](https://i.imgur.com/8LjNAmM.png)
##### Each little square represents the prevelance (the redder the more prevalent) of a frequency in 25ms of speech. The frequencies are graphed linearly.

Can you see the changes between the sounds 'sh', 'e', 'l', and 'a'?

One issue with the STFT here is that not all the frequencies are very relevant to us humans. Look at the frequencies above 4,000 Hz. They don't seem to carry quite as much information as those in the middle, as it pertains to speech.

Which is where the <a href="https://en.wikipedia.org/wiki/Mel_scale">Mel scale</a> comes in. 


![Imgur](https://i.imgur.com/mjtew0l.png)
#### Krishna Vedala [CC BY-SA 3.0 (http://creativecommons.org/licenses/by-sa/3.0/)], via Wikimedia Commons

The Mel scale puts into perspective human perception of frequencies: some frequencies we pay more attention to than others. (Would you be surprised if we paid more attention to frequency changes at the levels  of speech than those at the level of a screetching car?) 

By applying the mel scale, as well as the logarthimic scale (i.e. turning the (linear) power domain to the (non-linear) decibel domain), to the STFT, we get mel filterbank energies! 

### FBANK

![Imgur](https://i.imgur.com/Z3Xqbwa.png)
#### The focus is now on the frequencies relevant for speech. Note: the db are in respect to the highest db value, hence the highest == 0.

For deep learning algorithms, all of the forms above are useful in developing high performing speech classifiers (raw waveform, STFT, FBANK). We'll get into *why* in another post. But traditional machine learning algorithms (like support vector machines, random forests, hidden markov models-gaussian mixture models) don't handle such data as elegantly. There's simply a lot of features here for an algorithm to learn, and many of them <a href="https://en.wikipedia.org/wiki/Multicollinearity">colinear</a>. (Speech is just complicated man.)

### MFCC

Mel frequency cepstral coefficients (<a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">MFCC</a>) are basically filterbank energies (FBANK) with an additional filter applied: the <a href="https://en.wikipedia.org/wiki/Discrete_cosine_transform">discrete cosine transform</a> (DCT). This helps remove the colinearity prevalent in the FBANK features, making it easier for traditioal machine learning algorithms to actually learn the relevant features. (Deep learning algorithms are better at dealing with colinear features.)

![Imgur](https://i.imgur.com/CXWRmfw.png)

Based on some of the research I've read, if using FBANKs, one can use around 20 or 40 mel filters, and if using MFCCs, one can use 13 (12, if not including the 1st coefficient as it pertains to amplitude, not necessarily to speech sounds), 20, or 40 coefficients. This largely depends on whether you want to focus only on the speech sounds, i.e. 'd' vs. 'p' or 'e' vs 'u', which would mean you would use fewer MFCCs, 13 for example. For intonation, emotion, and other characteristics, you may want to explore using all 40 MFCCs. 

In this <a href="https://github.com/a-n-rose/Build-CNN-or-LSTM-or-CNNLSTM-with-speech-features">repo</a>, you can visually explore what different settings for feature extraction look like, and also see how they influence the training of different deep learning models. 

This is just an intro to get you comfortable with the features: raw waveform, STFT, FBANK, and MFCC, as these are very relevant in current research. As you continue exploring, you will most definitely come across others to add to your library of speech features.

I hope this has been helpful! 


## Further exploration:

Identifying depression in <a href="http://europepmc.org/backend/ptpmcrender.fcgi?accid=PMC3652557&blobtype=pdf">adolescents</a>

More in-depth feature extraction <a href="https://haythamfayek.com/2016/04/21/speech-processing-for-machine-learning.html">with Python code</a>.
