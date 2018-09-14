---
layout: post
title: "Upgrading my voice activity dectection (VAD) algorithm"
date: 2018-09-06
--- 

My Mimic Master Game is having trouble finding speech. As long as no background noise exists, it's fine, but with noise, sometimes it's spot on, other times it cuts off the mimic or holds onto several moments of silence/background noise. It is time for an upgrade. 

I will try integrating similar strategies as that of the author of <a href="https://github.com/marsbroshok/VAD-python/blob/master/vad.py">this repository</a>.

For some background, my first attempt involved the following. I collected the STFTs from the mimic and the noise signals. Then I calculated their powers and energy levels, as well as the variances of those. (You can see more details of the entire process <a href="https://a-n-rose.github.io/2018/08/24/mimic-master-pitchcurve-vs-fingerprint.html">here.</a>) For speech detection, I looked at the energy levels of the mimic data and if the energy levels were higher than that of the mean energy level plus the variance of the energy levels of the noise signal, and for several consecutive energy values, then at that point, speech began. (This would be reversed if searching for when speech ended.)

In other words, for each energy value in the mimic data:

<a href="https://www.codecogs.com/eqnedit.php?latex=e_i,&space;e_{i&plus;1},&space;e_{i&plus;2}&space;>&space;e_{noise&space;mean}&space;&plus;&space;e_{noisevariance}&space;\Rightarrow&space;speech" target="_blank"><img src="https://latex.codecogs.com/gif.latex?e_i,&space;e_{i&plus;1},&space;e_{i&plus;2}&space;>&space;e_{noise&space;mean}&space;&plus;&space;e_{noisevariance}&space;\Rightarrow&space;speech" title="e_i, e_{i+1}, e_{i+2} > e_{noise mean} + e_{noisevariance} \Rightarrow speech" /></a>

###### e stands for 'energy'

For now I am educating myself on the above mentioned repository. This post will be updated soon. 

...

After looking a bit more into voice detection, I decided to keep my simple strategy but I improved it to show directly that it could indicate when sound (i.e. speech) was present. I also simplified it further, removing any noise values. I simply examined whether or not the energy in a given sample was greater than the mean energy of the signal, for three consecutive samples. (A similar strategy of comparing energy levels within the signal was conducted by the author of the repository from <a href="https://github.com/marsbroshok/VAD-python/blob/master/vad.py">above</a>.)

Unlike that author, I did not attempt to remove silences throughout a recording as that is not my goal (as of yet). I am only interested in removing the beginning and ending silences of a recording.

These are the plots when this script was applied to a cat meowing:

#### Original signal
![Imgur](https://i.imgur.com/mfTW3tx.png)

#### Signal with silences removed
![Imgur](https://i.imgur.com/T1d29HD.png)

#### Original signal energy
![Imgur](https://i.imgur.com/WfwyDiv.png)

#### Signal energy with silences removed
![Imgur](https://i.imgur.com/KHbb3Ke.png)

#### Original signal power
![Imgur](https://i.imgur.com/k76a8Rk.png)

#### Signal power with silences removed
![Imgur](https://i.imgur.com/Zhk7ykt.png)


Haha, please forgive the titles of those graphs. Perhaps they're a bit too long. These were made via matplotlib.

I am happy with the results so far, and to see that my simply implementation and original methodology wasn't too far off. 
