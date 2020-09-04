---
layout: post
title: "Augmenting Speech Data for Deep Learning"
date: 2020-08-11
published: false
--- 

For speech and sound augmentation, there are a few available tools one can use. 

<a href="https://github.com/makcedward/nlpaug">NLPAUG</a> applies a few augmentations to sound data (as well as other kinds of language augmentation, like text); one can apply these augmentations to deep learning models in TensorFlow or PyTorch (and perhaps others).

<a href="https://ai.googleblog.com/2019/04/specaugment-new-data-augmentation.html">SpecAugment</a> is a creation of Google Brain that looks really cool. It is a computationally efficient way of augmenting sound and speech by applying augmentatons directoy to the spectrogram, rather than the waveform itself.

I think that for real-world models, these are probably great tools to use. But for the purpose of exploring sound data in the world of deep learning, they don't offer very much wiggle room for experimentation. 

For example, SpecAugment applies augmentation to mel spectrograms. That is great if that is the type of features you want to train your model with but a bit problematic if you would like to train instead with raw sound data in the time (signal samples) or frequency (short-time Fourier transform) domain. 

Additionally, in research there are multiple ways one can transform sound and speech data, several of which are not included in these tools. For those who want to try out different techniques to better grasp how augmentation influences the data and resulting model, perhaps building your own augmentation techniques is simply what you're into at the moment (I know I am).

In SoundPy, that is pretty much the goal. It isn't to make the best augmentation techniques or to make it the most computationally efficient as possible. It is to explore techniques applied in research and see what they look like with various datasets and model architectures. It also may serve as testing ground to see if further implementation of such methods is worth while.


# Purpose

Speech and sound augmentation is not yet incorporated in TensorFlow.

How should augmentation best be applied to speech data? Is this realistic, given large datasets and ability to augment data on the fly?

What if one wants to augment and train a model, as features are extracted?

Does it matter significantly if augmentations are fed to the model in a random order? (e.g. applied as the features are extracted beforehand, and then randomized.)

Or can data be randomized and augmented with a type of augmentation, over and over, as it is fed to the model? (e.g. first application of random noise to all data, then pitch shift, etc.)

What are some examples of how people have implemented augmentation techniques and speech / sound data?

# Issues

## Extract Features Before or During Training

Do you want to extract features before training or during training?

If during, it may take longer, but no space for the extracted features is wasted.

How have others extracted features during training?

