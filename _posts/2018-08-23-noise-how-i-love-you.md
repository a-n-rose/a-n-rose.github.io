---
layout: post
title: "Noise: How it saved my applications"
date: 2018-08-23
--- 

I never thought I would like noise so much. All it took though, like anything really, was a bit of playing around. 

Two recent projects of mine recorded speech and assessed it somehow, either by comparing it with a target recording (<a href="https://a-n-rose.github.io/2018/08/01/mimic-master.html">Mimic Master</a>) or by classifying it with an ANN classifier (<a href="https://a-n-rose.github.io/2018/08/22/language-classifier.html">Language Classifier</a>).

Before embarking on these projects, I knew that different cellphones and computers probably had different microphones but I didn't recognize, or never really thought about, how much just those differences alone would influence 'background noise' of a recording. This <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5426841/pdf/sensors-17-00917.pdf">article</a> explores such differences. 

Steps I took to reduce influences of varying microphone standards and actual environmental noise include:
* developed noise subtraction functions***
* developed speech detection functions*
* adjusted volume of recorded speech to match target recordings*
* removed the very beginning of recordings to remove clicks that might be there*
* scaled speech data with scikit-learn**

###### * Mimic Master 
###### ** Language Classifier
###### *** Both Mimic Master and Language Classifier


