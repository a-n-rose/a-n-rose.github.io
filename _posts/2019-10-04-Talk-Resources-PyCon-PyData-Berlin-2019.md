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

## Notebooks.ai

I have a few Jupyter notebooks with accompanying sound data and necessary code (package and modules) loaded at Notebooks.ai: <a href="https://notebooks.ai/a-n-rose">Aislyn's Sound Playground</a> (Recently some labs weren't loading, so if you encounter problems.. you're not alone..)

## GitHub

If you would like to access the notebooks on their own, you can find them at <a href="https://github.com/a-n-rose/experiment-with-sound-in-Python-filtering-and-deep-learning">this repo</a>. The notebook 'create_sound' should work without the sound data or code; I aim to get the others working on <a href="https://hub.gke.mybinder.org/user/a-n-rose-experi-d-deep-learning-l9fqpls1/tree">Binder</a> as well as on Notebooks.ai, but can't say exactly when I'll be able to do this.

# Problems using your own sound?

They are probably not compatible with the software/ online environment. This <a href="https://github.com/a-n-rose/python-sound-prep#prepare-audio-for-jupyter-lab">repo</a> should be able to help with that. As of now the software uses <a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.io.wavfile.read.html">scipy.io.wavfile</a>, which accepts only .wav files with bitdepth 16 and 32. That was the library I could use for reading soundfiles into JupyterLab. Sorry for the inconvenience!

# Visualization

If you would like to visualize sound (plot or .png file), you can do that with <a href="https://github.com/a-n-rose/python-sound-prep#visualizing-sound">this resource</a>. Visualize the sound in the time domain (sound wave) and in the frequency domain (MFCC and FBANK features)

# Datasets

A collection of <a href="https://a-n-rose.github.io/2019/01/06/resources-publicly-available-speech-databases.html">publicly available datasets</a>

# Background Info

## Filtering / Digital Signal Processing

This book, <a href="https://www.crcpress.com/Speech-Enhancement-Theory-and-Practice-Second-Edition/Loizou/p/book/9781138075573">Speech Enhancement: Theory and Practice, by P.C. Loizou</a> was somewhat approachable with respect to handling sound and filtering it for enhancing speech, especially in applications of code.

## Deep Learning / Architecture of Convolutional Neural Networks etc.

The <a href="http://www.deeplearningbook.org/">Deep Learing Book</a> was also very helpful in increasing understanding of CNNs in an accessible manner.
