---
layout: post
title: "Interactive Digital Signal Processing in Jupyter"
date: 2019-08-16
---

Update (Oct. 12, 2019) Here is my repo called <a href="https://github.com/a-n-rose/Python-Sound-Tool">PySoundTool</a> with various tools for visualizing sound, training sound classifiers, etc. Additionally, you can access Jupyter notebooks on <a href="https://mybinder.org/v2/gh/a-n-rose/Python-Sound-Tool/master">mybinder</a>, without needing an account. It may take a few minutes to load as I include a small amount of audio data for you to experiment with. They offer basically the same functionality as the notebooks I describe below on notebooks.ai. 

(On to the older post)

I put together a <a href='https://notebooks.ai/a-n-rose/working-with-signals-c2032035'>Jupyter Lab notebook</a> for creating and analyzing signals in Python. I try to make the math behind that of signal creation, the <a href='https://whatis.techtarget.com/definition/Nyquist-Theorem'>Nyquist Theorem</a>, and the fast <a href='https://en.wikipedia.org/wiki/Fourier_transform'>Fourier Transform</a> (FFT) a little bit more accessible. 

Note: this notebook should be more of an aid to additional materials about digital signal processing, especially that of the FFT. My goal is to offer a perspective of some components of the FFT that I found missing when researching it myself; additionally, the Jupyter notebook offers the opportunity to directly see how frequency, noise, sample rate, etc. influence digital signal processing.

Upcoming events: 

September 2nd we present our smart noise filter prototype <a href='https://prototypefund.de/en/projects/round5/'>NoIze</a>. 

September 10th we will give a workshop for PyLadies Berlin, <a href='https://www.meetup.com/PyLadies-Berlin/events/263676106/'>centering on our open source smart filter</a>.

October 11th we will give a talk at PyCon / PyData Berlin called <a href='https://www.youtube.com/watch?v=BJ0f2x49Imc&feature=youtu.be'>Take control of your hearing: Accessible methods to build a smart noise filter</a>
