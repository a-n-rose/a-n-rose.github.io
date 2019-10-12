---
layout: post
title: "Collection of Functions for Working with Sound in Python"
date: 2019-10-01
---

I enjoy working with sound. I assume if you are reading this you do as well. 

I have been having a lot of fun building interactive <a href='https://notebooks.ai/a-n-rose'>Jupyter notebooks</a> on notebooks.ai (also available on <a href="https://mybinder.org/v2/gh/a-n-rose/Python-Sound-Tool/master">binder</a> - it just needs a couple minutes to load). One problem I came across was a limited number of Python libraries I could use within the notebooks.

When working on my local computer, the library I turn to most often to handle sound data is <a href="https://librosa.github.io/librosa/">Librosa</a>. I feel like it has accompanied me on my entire sound journey and will well into the future. One example: when loading sound files with Librosa, it automatically scales the sound to their respective volumes, as well as keeping them normalized between -1 and 1. I have found loading sounds with other libraries a tad more difficult (sound may not be scaled or normalized) and require more involvement. Which in the long run is probably a good thing.

Fun fact, the <a href="https://aislynrose.bitbucket.io/">smart noise filter</a> I built originally used Librosa to load sound files up until I tried using it in a Jupyter notebook. I then switched it last minute to one that did work in Jupyter notebooks: <a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.io.wavfile.read.html">Scipy.io.wavfile's read</a> and write functionality. 

There are still some kinks I'm working out with that. For example, as of now, unhandled errors get spit out when someone tries feeding an mp3 file into the program. Even more confusing, it also can't handle wav files if the bit depth is 24. Improving those bugs are on my ToDo list..

Long story short, I've started collecting some of the functions I've built, not only for implementing sound filtering and such within Jupyter notebooks, but also adding sounds together for general experimentation.

For reference, here is the <a href="https://github.com/a-n-rose/Python-Sound-Tool#sound-file-prep">repository</a> where you can read through the README and look through the code for examples of how to use the functions depending on your needs.

In the future I will add visualization tools to aid understanding and familiarity with how windowing and feature extraction affects sound data for filtering or deep learning.

Other than that, happy October! 
 
