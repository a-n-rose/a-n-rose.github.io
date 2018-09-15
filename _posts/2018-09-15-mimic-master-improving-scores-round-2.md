---
layout: post
title: "Mimic Master - Second Attempt: I'm a Mimic Master!!!"
date: 2018-09-15
---

I did it! I improved on my <a href="/2018/08/24/mimic-master-pitchcurve-vs-fingerprint.html">Mimic Master</a> game. It actually works now!

Changes I made include the following.

## Voice Activity Detection (VAD)

I altered my <a href="/2018/09/06/updating-VAD.html">VAD function</a> to use the mean square of the short-time Fourier transform (STFT) values to indicate energy, instead of the root mean square. That allowed energy differences in the signal to be a bit more obvious, quite important when detecting speech amongst background noise. 

I also did not compare the energy levels of the signal with those of the background noise to detect speech; I simply compared the mean energy values and a specific energy value of the signal. If the specific energy value was greater than the mean, for 3 consecutive values, then speech was marked as present. I only considered background noise for the noise reduction function.

## Calculating Similarity

The advancements I made in the scoring system can be viewed in detail <a href="/2018/08/29/comparing-prosody.html">here</a>. 

Basically, instead of using the Pearson's correlation coefficient, I implemented Hermes weighted correlation coefficient, following suit of a paper also comparing prosody of two signals. Additionally (also in light of that paper), I collected pitch values in longer windows, not just of single data points. This allowed for similar pitch productions to be a little off in timing. The pitch-curves don't have to align perfectly anymore in order to get a high score (which was the case before).

The final score was the difference between the similiarity scores of the mimic to the target and the background noise to the target. The absolute value of that difference was then multiplied by 100. This seemed to counteract imperfections in the similarity measures: the score was not so dependent on how very similar it was to the target (some kinds of sounds were easier to calculate via pitch curve, thus earned higher similarity ratings; for example, a snorting pig was not as easy to compare (or mimic) as a hooting owl), rather how much better the mimic was in comparison to silence. 

## Where I saw the differences

With the above changes, I could finally play the game and feel good about the scoring system. It seemed pretty much fair. Also, when I checked the manipulated wavefiles (noise reduction; volume matching; beginning and ending silence removal), those had also vastly improved, particularly in identifying when a mimic started and stopped - except for when I mimicked a cat meowing. It had a hard time identifying when I stopped meowing.

That being said, there is of course room for improvement. The game could compare some sounds better than others; for example, I didn't perform well with most of the bird sounds, except for the owl and rooster. Oh and I failed mimicking cats, despite having had cats and mimicked them avidly as a kid. I suspect the larger amounts of background noise present in the target recordings reduced the ability of the VAD to identify the mimic from the larger amounts of background noise. So, to avoid that, *sounds used for mimicking should not have significant amounts of extended silences.*

## Next Steps

I think with additional measures, for example acoustic fingerprints (I will explore developing my own), would likely help the scoring improve even more.

Also, I will eventually implement a measure to avoid including too much background silence into the calculation of how long users have to mimic a sound. Right now, the user has a couple more seconds to mimic than the length of the target sound; problem there is, if the original sound has a lot of silence, the user's mimic will have a lot of unnecessary background noise to process. 

Here is a rundown of the improved game.

1) First background noise is collected. 

### Background noise 
![Imgur](https://i.imgur.com/utuvQcC.png?1)
#### 5 seconds of my apartment, recorded with my computer

2) Then a sound is played:

### Target sound: 
![Imgur](https://i.imgur.com/ISfq9bB.png?1)
#### Mooing cow

3) The game records the user for the duration of the target sound, plus a couple of seconds to account for user response delay.

### User mimic: 
![Imgur](https://i.imgur.com/jofKraq.png?1)
#### Mooing cow original

### User mimic: 
![Imgur](https://i.imgur.com/ysJNwr0.png?1)
#### Mooing cow noise reduced

### User mimic: 
![Imgur](https://i.imgur.com/7XEMYLp.png?1)
#### Mooing cow volume matched (to the target) and silences removed
 
4) Points earned: 27


Repeat! (Except for the collection of background noise)


1) A sound is played:

### Target sound: 
![Imgur](https://i.imgur.com/sEU5R43.png?1)
#### Hooting owl

2) The game records the user for the duration of the target sound, plus a couple of seconds to account for user response delay.

### User mimic: 
![Imgur](https://i.imgur.com/PtXSyVR.png?1)
#### Hooting owl original

### User mimic: 
![Imgur](https://i.imgur.com/97hcZBd.png?1)
#### Hooting owl noise reduced

### User mimic: 
![Imgur](https://i.imgur.com/SwfInqS.png?1)
#### Hooting owl volume matched (to the target) and silences removed

3) Points earned: 113

Wow! Not bad..

Total points collected: 140

Repeat!

1) A sound is played:

### Target sound: 
![Imgur](https://i.imgur.com/Y8kEXJq.png?1)
#### Meowing kitten

2) The game records the user for the duration of the target sound, plus a couple of seconds to account for user response delay. (See why the extra silence is a problem?)

### User mimic: 
![Imgur](https://i.imgur.com/75nnFqQ.png?1)
#### Meowing kitten original

### User mimic: 
![Imgur](https://i.imgur.com/S2sfX68.png?1)
#### Meowing kitten noise reduced

### User mimic: 
![Imgur](https://i.imgur.com/TsUxxSb.png?1)
#### Meowing kitten volume matched (to the target) and silences removed

3) Points earned: 17

Frowny face.

Well, from the above visuals, you can see that the volume matching works better on some files than on others. You can barely see my kitten mimic after the volume has been matched. Also, it's clear that silences in target sounds should be removed to improve the comparison analyses. 

All in all though, it's a pretty fun and successful game!

Hope this was interesting/helpful/useful! Have a great day and thanks for reading. 

P.S. Feel free to write me for questions/tips/suggestions. 

###### All signal visuals created in Audacity
