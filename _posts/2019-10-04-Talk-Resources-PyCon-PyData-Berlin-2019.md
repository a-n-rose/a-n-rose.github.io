---
layout: post
title: "PyCon DE and PyData Berlin 2019 Talk Resources"
date: 2019-10-10
---

# Aislyn's resources for <a href="https://de.pycon.org/program/pydata-jzw9he-take-control-of-your-hearing-accessible-methods-to-build-a-smart-noise-filter-peggy-sylopp-aislyn-rose/">the talk</a>:

## **Take control of your hearing: Accessible methods to build a smart noise filter**

# Smart noise filter

The repository for the smart noise filter functionality I built: <a href="https://github.com/pgys/NoIze">NoIze</a>

# Sound Playground

## GitHub repo

I have put together a tool called <a href="https://github.com/a-n-rose/Python-Sound-Tool">PySoundTool</a> for visualization, filtering, sound creation, training sound classifiers etc. 

## Notebooks.ai

I have a few Jupyter notebooks with accompanying sound data and necessary code (package and modules) loaded at Notebooks.ai: <a href="https://notebooks.ai/a-n-rose">Aislyn's Sound Playground</a> (Recently some labs weren't loading, so if you encounter problems.. you're not alone..)

# Problems using your own sound?

They are probably not compatible with the software/ online environment. This <a href="https://github.com/a-n-rose/Python-Sound-Tool#convert-soundfiles-for-use-with-scipyiowavfile">repo</a> should be able to help with that. (See subsection 'Convert Soundfiles for use with scipy.io.wavfile' in the README.)

Previously the software used Librosa to load sound files but because that library could not be imported into Jupyter environments, the software now uses a Jupyter friendly library: <a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.io.wavfile.read.html">scipy.io.wavfile</a>. This module accepts only .wav files with bitdepth 16 and 32. The software only accepts mono channel sound as well (something Librosa did automatically). Sorry for the inconvenience! 

# Visualization

If you would like to visualize sound (plot or .png file), you can do that with <a href="https://github.com/a-n-rose/Python-Sound-Tool#visualization">this resource</a>. Visualize the sound in the time domain (sound wave) and in the frequency domain (MFCC and FBANK features)

# Datasets

A collection of <a href="https://a-n-rose.github.io/2019/01/06/resources-publicly-available-speech-databases.html">publicly available datasets</a>

# Background Info

## Filtering / Digital Signal Processing

This book, <a href="https://www.crcpress.com/Speech-Enhancement-Theory-and-Practice-Second-Edition/Loizou/p/book/9781138075573">Speech Enhancement: Theory and Practice, by P.C. Loizou</a> was somewhat approachable with respect to handling sound and filtering it for enhancing speech, especially in applications of code.

## Deep Learning / Architecture of Convolutional Neural Networks etc.

The <a href="http://www.deeplearningbook.org/">Deep Learning Book</a> was also very helpful in increasing understanding of CNNs in an accessible manner.
