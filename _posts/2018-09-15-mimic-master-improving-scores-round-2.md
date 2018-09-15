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

The advancements I made in the scoring system can be viewed <a href="/2018/08/29/comparing-prosody.html">here</a>. Basically, I collected pitch values in longer windows, not just of single data points. This allowed for similar pitch productions to be a little off in timing. The pitch-curves don't have to align perfectly in order to get a high score (which was the case before).

Lastly, the score was calculated not soley based on a coefficient of similarity between the mimic and the target pitch curves. Rather, the score was calcuated based on the difference between how similar the mimic was to the target and how similar background noise was to the target. 

## Where I saw the differences

With the above changes, I could finally play the game and feel good about the scoring system. It seemed pretty much fair. Also, when I checked the manipulated wavefiles (noise reduction; volume matching; beginning and ending silence removal), those had also vastly improved, particularly in identifying when a mimic started and stopped - except for when I mimicked a cat meowing. It had a hard time identifying when I stopped meowing.

That being said, there is of course room for improvement. The game could compare some sounds better than others; for example, I didn't perform well with most of the bird sounds, except for the owl and rooster. Oh and I failed mimicking cats, despite having had cats and mimicked them avidly as a kid. I suspect the larger amounts of background noise present in the target recordings reduced the ability of the VAD to identify the mimic from the larger amounts of background noise. So, to avoid that, *sounds used for mimicking should not have significant amounts of extended silences.*

## Next Steps

I think with additional measures, for example acoustic fingerprints (I will explore developing my own), would likely help the scoring improve even more.

Also, I will eventually implement a measure to avoid including too much background silence into the calculation of how long users have to mimic a sound. Right now, the user has a couple more seconds to mimic than the length of the target sound; problem there is, if the original sound has a lot of silence, the user's mimic will have a lot of unnecessary background noise to process. 

I will update this post with visuals of how this game processed the audio files and generated scores so that the improvements are a bit easier to see. 

Hope this was interesting/helpful/useful! 

Have a great day and thanks for reading. 

P.S. Feel free to write me for questions/tips/suggestions. 
