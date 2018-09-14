---
layout: post
title: "Building Multi-Lingual Speech to Text Application"
date: 2018-09-12
---

I have used Voxforge's speech data to play around with ANN and LSTM models and how they could be trained to identify a certain language spoken. They also have a lot of annotated speech, which I would like to use to develop a multilingual speech to text application.

First I will further examine how the speech files are organized for each language, made avaiable by Voxforge. 

# Speech Database Organization

## German

For German, all of the speech data can be downloaded in a zipfile. Within this zipfile, three directories, 'train', 'test', and 'dev' hold numerous WAV files as well as XML files. The XML files correspond to a set of WAV files; that set of WAV files seem to be various versions of the same recording (preprocessed differently). The annotations of what a speaker said in those WAV files are in the XML files. I will use Python's <a href="https://docs.python.org/2/library/xml.etree.elementtree.html#module-xml.etree.ElementTree">ElementTree XML API</a> to access the information. 

```
import xml.etree.ElementTree as ET

file_xml = "./German/2014-10-10-13-22-36.xml"
tree = ET.parse(file_xml)
root = tree.getroot()
```
After playing around a bit, I found that the following revealed relevant data from this specific file:
```
speaker_sex = root[3].text
speech_text = root[6].text #perhaps also 7
speech_type = root[8].text #command, request, question etc.
speaker_region = root[10].text #where the speaker comes from
```

Let's pop into another XML file and see if these indices correspond to the same data.

```
file_xml = "./German/2014-10-10-13-22-24.xml"
tree2 = ET.parse(file_xml)
root2 = tree2.getroot()
```
Yup!


## English

The English annotation files are in a separate folder from the WAV files. The English speech has been organized as such: for each set of recordings (one speaker can complete more than one), there are two directories organizing the information: a 'wav' directory, with the WAV files, and a 'etc' directory, with various files containing text documents, one of which a 'PROMPTS' file. In this file, the titles of the WAV files are included, along with the text spoken by the speaker. 

# Ideas:
* Perhaps use <a href="http://espeak.sourceforge.net/languages.html">Espeak</a> to translate text to IPA? This could allow for languag teaching applications: show how to pronounce various words in the relevant language. Communicate with others thru different languages
* Remove silences from speech
* Create pictures of whole- mel-log-spectrograms and train with IPA sequences (do these need to be same length?)
* Can I take into account length of IPA segment and length of speech, divide them accordingly, with windows, and supply the little window of spectrogram as input and output two or three IPA letters, that should correspond to the segment of speech after silence removal?
