---
layout: post
title: "Mimic Master: Learning how to use voice biometrics and speech analysis with shoddy recordings (i.e. lots of background noise)"
date: 2018-08-01
---

All this game does is play a recording and then records the user, ideally mimicking what was just played. After each recording, a score is calculated based on how well the user mimicked the sound. Easy right? 

Maybe for some but I'm still working on this one. While this is still a work in progress, you can try out the best working version <a href = "https://github.com/a-n-rose/mimic-master-how-well-can-you-mimic">here.</a>

I have come across several problems, ranging from microphone quality to comparing spectral fingerprints. 

Any app that records a user remotely has to deal with varying microphone quality and background noise levels. My first challenge was finding a simple way to cancel out whatever background noise a user might have due to those variables. I attempted to recreate the noise reduction technique Audacity performs very well; I would say I was relatively successful as background noise was removed from the user's mimics, which improved the analysis of their speech. In the game, I started out by testing the user's mic by recording 5 seconds of their background noise. I calculated the spectral power in that recording and subtracted it from all of the user's subsequent mimics. One way I knew this worked was testing out my game while my vacuuming robot was on, right next to me (I had somehow ignored the little guy - we call him Roby). I was shocked to find basically silent speech recordings! So, needless to say, this game shouldn't be played while vacuuming your apartment.

A surprise issue I faced was oddities in the recordings throughout the game: it seemed the first recording the game took, artifacts in the first milliseconds disrupted analyses of either background noise or the user's speech. Similar patterns were found in the course of this research <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5426841/pdf/sensors-17-00917.pdf">project.</a> As a result, I decided to remove the first few milliseconds of every recording the game took, which improved analyses. (Note: since I was not sure if these artifacts would always only occur in the first recording, but perhaps also the third or 20th one, the beginning was removed from all recordings.)

I am currently still working on how to best compare the target sound and the user's mimic. For now I am implementing a very simple comparison measure: the area beneath the pitch curves. This is far from perfect. First, I had to remove the silence from both the target sound and the user's recording to get the pitch curves to start at the same time. But if the user's timing is just slightly off throughout the mimic, that difference in pitch curve will greatly exaggerate the imperfection of the user's mimic. Therefore, other methods of similarity such as <a href="https://stackoverflow.com/questions/21647120/how-to-use-the-cross-spectral-density-to-calculate-the-phase-shift-of-two-relate">cross-spectral density</a> need to be implemented.

Another hurdle I'm trying to jump over is how to balance strict enough measures to ensure that if the user says nothing, that is clearly identified by the game, but also able to identify sublties in the user's recording that might get lost if the measures are too rigid. I'm hoping as I improve the comparison methods this will get better as a result. 

For the target sounds, I collected recordings from <a href="https://freesound.org/">freesound</a>, mainly animals 'cause those are the most fun. I am excited to eventually include different kinds of sounds, like famous speakers and non-living sounds. I'm sure many more fun issues will appear with those!




