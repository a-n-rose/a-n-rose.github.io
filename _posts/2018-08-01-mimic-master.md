---
layout: post
title: "Mimic Master: Learning how to use voice biometrics and speech analysis with shoddy recordings (i.e. lots of background noise)"
date: 2018-08-01
---

Ever wanted to improve your animal mimicking skills? Well, you're in the right place. While this is still a work in progress, you can try out the best working version <a href = "https://github.com/a-n-rose/mimic-master-how-well-can-you-mimic">here.</a>

This has been a blast and adventure to build. I have come across several problems, ranging from differencs in microphone quality to detecting speech amidst noise, let alone comparing pitch or prosody of two audio signals. 

Any app that records a user remotely has to deal with varying microphone quality and background noise levels. My first challenge was finding a simple way to cancel out whatever background noise a user might have due to those variables. 

I attempted to recreate the noise reduction technique Audacity performs very well; I would say I was relatively successful as background noise was removed from the user's mimics, which improved the analysis of their speech. In the game, I started out by testing the user's mic by recording 5 seconds of their background noise. I calculated the spectral power in that recording and subtracted it from all of the user's subsequent mimics. 

##### Functions defined (see <a href="https://github.com/a-n-rose/mimic-master-how-well-can-you-mimic">the repository</a> for the entire code): 
```
import sounddevice as sd

def record_user(duration):
    fs = 22050
    user_rec = sd.rec(int(duration*fs),samplerate=fs,channels=1)
    sd.wait()   
return(user_rec)

def test_mic(duration):
    user_rec = record_user(duration)
    sd.wait()
    if user_rec.any():
        sd.wait()
        print("Thanks!")
        return user_rec
    else:    
        print("Hmmmmm.. something went wrong. Check your mic and try again.")
        if start_game('test your mic'):
            test_mic(duration)
        else:
            return None

```            
##### Put those functions to work to get background noise.
```
duration = 5
print("\nThis next step will take just {} seconds\n".format(duration))
background_noise = test_mic(duration)

```

One way I knew this worked was testing out my game while my vacuuming robot was on, right next to me (I had somehow ignored the little guy - we call him Roby). I was shocked to find basically silent speech recordings! So, needless to say, this game shouldn't be played while vacuuming your apartment.

##### My Mimic with Vacuum in the Background
![Imgur](https://i.imgur.com/dgrsfSP.png)
##### My Mimic Post Noise Reduction:
![Imgur](https://i.imgur.com/XdiJLOD.png)
###### Visualizations created using <a href="https://www.audacityteam.org/">Adacity</a>

A surprise issue I faced was oddities in the recordings throughout the game: it seemed the first recording the game took, artifacts in the first milliseconds disrupted analyses of either background noise or the user's speech. Similar patterns were found in the course of this research <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5426841/pdf/sensors-17-00917.pdf">project.</a> As a result, I decided to remove the first few milliseconds of every recording the game took, which improved analyses. (Note: since I was not sure if these artifacts would always only occur in the first recording, but perhaps also the third or 20th one, the beginning was removed from all recordings.)
##### Example of Beginning Recording Click
![Imgur](https://i.imgur.com/aeqYaoM.png)

I am currently still working on how to best compare the target sound and the user's mimic. At first I tried using Chromaprint and generated a score based on how similar that software calculated the target and mimic to be. The results were disappointing though: chance. I could have screamed into the microphone when the target was a whispering panda, and then expertly mimicked a chimpanzee call, our closest animal relative, and ended up with comparable scores. 

Perhaps Chromaprint will work better as I improve how I preprocessing the audiofiles, or perhaps I missed something when I implemented it, but for now I've turned to a much more basic approach: Pearson correlation coefficient, i.e. measuring the area beneath pitch curves. This is far from perfect, though. First, I had to remove the beginning silence from both the target sound and the user's recording to get the pitch curves to start at the same time. But if the user's timing is just slightly off throughout the mimic, that difference in pitch curve will greatly exaggerate the imperfection of the user's mimic. Therefore, other methods of similarity such as <a href="https://stackoverflow.com/questions/21647120/how-to-use-the-cross-spectral-density-to-calculate-the-phase-shift-of-two-relate">cross-spectral density</a> or <a href="https://perso.limsi.fr/mareuil/publi/IS110831.pdf">Dynamic Time Warping</a> need to be implemented.

Another hurdle I'm trying to jump over is how to balance strict enough measures to ensure that if the user says nothing, that is clearly identified by the game, but also able to identify sublties in the user's recording that might get lost if the measures are too rigid. I'm hoping as I improve the comparison methods this will get better as a result. 

For the target sounds, I collected recordings from <a href="https://freesound.org/">freesound</a>, mainly animals 'cause those are the most fun. I am excited to eventually include different kinds of sounds, like famous speakers and non-living sounds. I'm sure many more fun issues will appear with those!




