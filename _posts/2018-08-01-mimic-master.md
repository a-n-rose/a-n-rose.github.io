---
layout: post
title: "Mimic Master: Learning how to use voice biometrics and speech analysis with shoddy recordings (i.e. lots of background noise)"
date: 2018-08-01
---

All this game does is play a recording and then records the user, ideally mimicking what was just played. After each recording, a score is calculated based on how well the user mimicked the sound. Easy right? 

Maybe for some but I'm still working on this one. 

I have come across several problems, ranging from microphone quality to comparing spectral fingerprints. While this is still a work in progress, you can try out the best working version <a href = "https://github.com/a-n-rose/mimic-master-how-well-can-you-mimic">here.</a>

A simple issue I faced was oddities as the game recorded: it seemed the first recording the game took, artifacts in the first milliseconds disrupted analyses of either background noise or the user's speech. Similar patterns were found in the course of this research <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5426841/pdf/sensors-17-00917.pdf">project.</a> As a result, I decided to remove the first few milliseconds of every recording the game took. This improved background noise as well as speech analyses.

I am currently still working on how to best compare the target sound and the user's mimic. For now I am implementing a very simple comparison measure: the area beneath the pitch curves. This is far from perfect. First, I had to remove the silence from both the target sound and the user's recroding to get the pitch curves to at least start at the same time. But if the user's timing is just slightly off, that difference in pitch curve will greatly exaggerate the imperfection of the user's mimic. Therefore, additional methods of similarity such as <a href="https://stackoverflow.com/questions/21647120/how-to-use-the-cross-spectral-density-to-calculate-the-phase-shift-of-two-relate">cross-spectral density.</a>

Another hurdle I'm trying to jump over is how to balance strict enough comparisons to ensure if the user says nothing, that is clearly identified by the game, but also able to identify sublties in the user's recording that might get lost if the measurements are too rigid. I'm hoping as I improve the comparison methods this will get better as a result. 

I collected sounds from <a href="https://freesound.org/">freesound</a>, mainly animals 'cause those are the most fun. I am excited to eventually apply different kinds of sounds, including famous speakers and non-living sounds. I'm sure many more issues will appear with those!




