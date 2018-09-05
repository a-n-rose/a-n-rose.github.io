---
layout: post
title: "Upgrading my Voice Activity Dectection Algorithm"
date: 2018-09-06
--- 

My Mimic Master Game is having trouble finding speech. As long as no background noise exists, it's fine, but with noise, sometimes it's spot on, other times it cuts off the mimic or holds onto several moments of silence/background noise. It is time for an upgrade. 

I will try integrating similar strategies as that of the author of <a href="https://github.com/marsbroshok/VAD-python/blob/master/vad.py">this repository</a>.

For some background, my first attempt involved the following. I collected the STFTs from the mimic and the noise signals. Then I calculated their powers and energy levels, as well as the variances of those. (You can see more details of the entire process <a href="https://a-n-rose.github.io/2018/08/24/mimic-master-pitchcurve-vs-fingerprint.html">here.</a>) For speech detection, I looked at the energy levels of the mimic data and if the energy levels were higher than that of the mean energy level plus the variance of the energy levels of the noise signal, and for several consecutive energy values, then at that point, speech began. (This would be reversed if searching for when speech ended.)

In other words, for each energy value in the mimic data:

<a href="https://www.codecogs.com/eqnedit.php?latex=e_i,&space;e_{i&plus;1},&space;e_{i&plus;2}&space;>&space;e_{noise&space;mean}&space;&plus;&space;e_{noisevariance}&space;\Rightarrow&space;speech" target="_blank"><img src="https://latex.codecogs.com/gif.latex?e_i,&space;e_{i&plus;1},&space;e_{i&plus;2}&space;>&space;e_{noise&space;mean}&space;&plus;&space;e_{noisevariance}&space;\Rightarrow&space;speech" title="e_i, e_{i+1}, e_{i+2} > e_{noise mean} + e_{noisevariance} \Rightarrow speech" /></a>

###### e stands for 'energy'

For now I am educating myself on the above mentioned repository. This post will be updated soon. 
