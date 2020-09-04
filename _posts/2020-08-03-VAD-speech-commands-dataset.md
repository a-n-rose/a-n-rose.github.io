---
layout: post
title: "Application of VAD to Purify Speech Datasets"
date: 2020-08-03
published: false
--- 

# Goal

How can I build a reliable VAD function?

How can I reliably remove bad data?

Could augmentation techniques make this less of an issue?


# Issues

## Speech Sounds 

Speech sounds have different qualities making identification of voice activity difficult. 


### VAD based off of research paper

Use three measures:
* energy
* frequency
* spectral flatness

### VAD based off of energy and filtering

If no VAD identified, run through wiener filter and try again

### VAD energy, then on the research (with and without filter)

If no VAD, run through more finicky one, with and without filter

### CPU usage with each of these approaches

And how much time it adds. Is it worth it?


### Using Speech Commands Dataset 

Which is better at addressing this issue? Current purpose: find corrupted or bad trianing data.

#### Does this help model training and performance? 

Which handles new speech better? With or without noise? Variable speakers?
