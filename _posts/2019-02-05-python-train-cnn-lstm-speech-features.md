---
layout: post
title: "Train a ConvNet, LSTM, or stacked ConvNet+LSTM with Various Speech Features"
date: 2019-02-06
---

(In the works..)

Have you messed around with Keras, perhaps using audio data? If you are anything like me, you had issues with dimensionality, among countless other things, but mainly dimensionality. Regardless, I hope you find this workshop helpful and who knows, maybe even soothing. 

The <a href="https://github.com/a-n-rose/Build-CNN-or-LSTM-or-CNNLSTM-with-speech-features">code</a> I put together allows you to extract several different speech/ sound features, and feed them to various networks. As the script is, you don't have to touch dimensionality. But, I suggest for you to have a peak and see how the dimensionality of your data needs to be adjusted to get that bugger of a model training. 

Where to start with such a complicted subject? 

Sound? 

Neural networks? 

Matrices?

I suppose we should actually start with a problem we'd like to solve. Let's say we want to expand speech recognition (where computers understand *what* we say ~ not necessarily *how* we say it) to identify other aspects of our speech signal. This can be useful (and is being explored) for health related apps, such as the tracking of depression or <a href="https://ac.els-cdn.com/S2352872915000160/1-s2.0-S2352872915000160-main.pdf?_tid=d04d9cc3-f992-4820-af98-f083c847c322&acdnat=1549408007_fe358db560e7df5618e8d68875824413">identifying the onset of Alzheimer's</a>.

Let's see what information is available in the raw waveform of speech:


![Imgur](https://i.imgur.com/sSYOMbq.png)
#### wave pulled from <a href="https://ai.googleblog.com/2017/08/launching-speech-commands-dataset.html">Speech Commands Dataset</a>


What information can we gather from this picture with our naked eye? 

We can see how much energy is in the signal, and when different speech sounds are made. The beginning fuzzy part looks like it matches to the sound 'sh'. But can we answer these questions: Which exact sounds is this person saying? Could I tell 'sh' from 'z' if I didn't know already that the word is 'Sheila'? Is this person male, female, a child or an adult? Which language are they speaking? And related to our question, is this speech typical or atypical, as in, is this person experiencing the onset of Alzheimer's?

Let's zoom in to see what the actual waves look like; maybe that will tell us something.

![Imgur](https://i.imgur.com/zWqjgkz.png)

Was this what you expected? These waves are pretty squiggly. Compare these waves with those below. 

![Imgur](https://i.imgur.com/VjNz6Pj.png)

What's the difference? Simply put, the sine wave has just one frequency; the speech sound wave has multiple frequencies, packed together. For us humans, and most standard machine learning algorithms, it is very hard to tell which frequencies are in there. 

This is where the Fourier Transform comes in!! 

For a really great explanation of the Fourier Transform, as well as some enjoyable visuals, watch this <a href="https://www.youtube.com/watch?v=spUNpyF58BY">video</a>.

Long story short, by applying this equation - equipped with several spiraling sinewaves, all with different frequencies - to the sound wave, it basically unpacks the frequencies from the sound wave. This can tell us the frequencies in the speech.

Why would frequencies be helpful? Think of how you tell if a speaker is a man, woman, or child. The most obvious way is hearing how high or low the speech is. That's one thing frequencies can tell us but they can potentially reveal additional characteristics, such as health.

Note: my background is more focused on clinical speech, so I will continue with the standards I've seen for handling speech; I can't (yet) speak for the standards for music or other realms of digital signal processing.

Anyways, while the Fourier Transform is amazing, it expects the signal to be unchanging. Think of the humming comming from your computer, or refrigerator. (It's my microwave's ringing that drives me bonkers.) Those signals stay relatively the same over time.

Speech is vastly different. The signal changes in the realm of milliseconds! These changes reflect the different speech sounds, or phonemes, we make. So, how do we find out the frequencies within such a varying signal? 

Apply the Fourier Transform in tiny little windows, specifically, the <a href="https://ccrma.stanford.edu/~jos/sasp/Short_Time_Fourier_Transform.html">short-time fourier transform</a> (STFT)

Most research papers I've come across apply the FT (I'm tired of typing that out) in windows of 25 milliseconds, meaning they 'transform' the signal at only 25 ms at a time. In order to catch variations in frequency that might happen within that 25 milliseconds, the FT is calculated in shifts of 10 milliseconds, going down the signal until all of it has been 'transformed'. In the future I will refer to this as a window size of 25 ms and a window shift of 10 ms. 


Knowing this, let's compare the raw waveform and it's transformed state:

![Imgur](https://i.imgur.com/sSYOMbq.png)


![Imgur](https://i.imgur.com/8LjNAmM.png)
##### Each little square represents the prevelance (the redder the more prevalent) of a frequency in 25ms of speech. The frequencies are graphed linearly.

Can you see the changes between the sounds 'sh', 'e', 'l', and 'a'?

One issue with the STFT here is that not all the frequencies are very relevant to us humans. Look at the frequencies at the bottom (0-128 Hz) and those at the very top (4,000 Hz+). They don't seem to carry quite as much information as those in the middle. 

Which is where the <a href="https://en.wikipedia.org/wiki/Mel_scale">Mel scale</a> comes in. 


![Imgur](https://i.imgur.com/mjtew0l.png)
#### Krishna Vedala [CC BY-SA 3.0 (http://creativecommons.org/licenses/by-sa/3.0/)], via Wikimedia Commons

The Mel scale puts into perspective human perception of frequencies: some frequencies we pay more attention to than others. (Would you be surprised if we paid more attention to frequency changes in the levels  of speech than those at the level of a screetching car?) 

By applying the mel scale, as well as the logarthimic scale (i.e. turning the (linear) power domain to the (non-linear) decibel domain), to the STFT, we get mel filterbank energies! 

![Imgur](https://i.imgur.com/Z3Xqbwa.png)
#### The focus in on the frequencies relevant for speech. Note: the db are in respect to the highest db value, hence the highest == 0.

(We're almost through the features, I promise.)

For deep learning algorithms, all of the forms above are useful in developing high performing speech classifiers. We'll get into *why* later. But traditional machine learning algorithms don't handle such data as elegantly. There's simply a lot of features here for an algorithm to learn, and many of them <a href="https://en.wikipedia.org/wiki/Multicollinearity">colinear</a>. (Speech is just complicated man.)

Mel frequency cepstral coefficients are filterbank energies (FBANK) with an additional filter applied: the <a href="https://en.wikipedia.org/wiki/Discrete_cosine_transform">discrete cosine transform</a> (DCT). Basically this removes the colinearity prevalent in the FBANK features. 

![Imgur](https://i.imgur.com/CXWRmfw.png)

For speech research, if using FBANKs, one usually uses 20 or 40 mel filters, and if using MFCCs, one usually uses 13 (12, if not including the 1st coefficient as it pertains to amplitude, not necessarily to speech sounds), 20, or 40 coefficients. This largely depends on whether you want to focus only on the speech sounds, i.e. 'd' vs. 'p' or 'e' vs 'u', which would mean you would use fewer MFCCs, 13 for example. For intonation, emotion, and other characteristics, you may want to explore using all 40 MFCCs.

Phew! Let's move on.

