---
layout: post
title: "Building Multi-Lingual Speech to Text Application"
date: 2018-09-12
---

I have used Voxforge's speech data to play around with ANN and LSTM models and how they could be trained to identify a certain language spoken. They also have a lot of annotated speech, which I would like to use to develop a multilingual speech to text application.

The annotations are in XML files so I will use Python's <a href="https://docs.python.org/2/library/xml.etree.elementtree.html#module-xml.etree.ElementTree">ElementTree XML API</a> to access the information. 

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

Idea: perhaps use <a href="http://espeak.sourceforge.net/languages.html">Espeak</a> to translate text to IPA? This could allow for languag teaching applications: show how to pronounce various words in the relevant language. Communicate with others thru different languages

