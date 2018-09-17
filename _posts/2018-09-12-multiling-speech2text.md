---
layout: post
title: "Building Multi-Lingual Speech to Text Application"
date: 2018-09-12
---

I have used Voxforge's speech data to play around with ANN and LSTM models and how they could be trained to identify a certain language spoken. They also have a lot of annotated speech, which I would like to use to develop a multilingual speech to text application.

First I will further examine how the speech files are organized for each language, made available by Voxforge. 

## Speech Database Organization

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

## Next Steps

Having looked a bit into the annotation data, I thought about how I could use it/how I wanted to use it. I decided to see how I can train a network with MFCC data and the annotations I have available. Already I know that the results won't be ideal, as the annotations are not aligned exactly with the speech productions. However, I want to see if any useful classifications/sequences can be learned with approximations of text and speech data. Furthermore, I also think that using mel-log-spectrograms of speech data would likely work better than MFCCs, but for now I will work with the data I have available: MFCCs of lots of English data, with beginning and ending silences removed.

To see if anything along these lines is achievable, I will focus on the English data only, as I am a native speaker of English and as I have already played a bit around with translating English to the international phonetic alphabet (IPA), which is what I want to do. My logic goes: translate the language to IPA, which should remove some characters which may not quite represent the time it takes to make a sound (e.g. in German 'sch' - pronounced 'sh' as in 'shoe' - and 'dsch' - pronounced 'j' as in 'jungle' - are sounds with a lot of letters that can be represented in IPA with fewer characters: ʃ  and dʒ, respectively). 

After removing extra characters, and also to be pronounceable as long as one knows the IPA, I will then correlate the length of the IPA text and that of the speech production and break them up in smaller segments. These segments, particularly of the speech, will overlap, in aims of catching more consisten patterns than strict segmentation allows for. The model trained will be a LSTM, that will produce sequences as output, not a single output, as I did for my <a href="/2018/09/07/language-classifier-LSTM.html">language classifier</a> project.

## Organizing data

In writing the code to translate the anntations to IPA, I have to think about how I want to store the data. Right now I like using SQLite3 as it can be used with Python without extra installations, making it easier to share with others. So, I'll stick with that for now. When collecting the annotations, all I need are the recording session ID (speakers could have more than one session), wavefile names, the orignal text, the IPA text, and the language type (English, German, etc.). 

```
import sqlite3
from sqlite3 import Error

database='speech2IPA.db'
tablename='speech2IPA'

conn = sqlite3.connect(database)
c = conn.cursor()

c.execute("CREATE TABLE IF NOT EXISTS {}(session_ID TEXT, filename  TEXT, annotation_original TEXT, annotation_IPA TEXT, label TEXT) ".format(tablename))
conn.commit()
```

Now, in order to collect the annotations, given how the information was stored, I use glob to navigate through files, and Path to collect the wavefile information within a path. Python can read text files so I don't need any imports for actually collecting the text within the files. (I used the knowledge about how the files were organized in the database to do this; see the section titled 'English')

```
import glob
from pathlib import Path
from subprocess import check_output

def annotations2IPA(c,conn,tablename):
    for session in glob.glob("**/*/"):
        files = Path(session)
        #the language label should be the first folder
        language = files.parts[0]
        #collect Annotations:
        annotation_file = session+'etc/PROMPTS'
        annotations = open(annotation_file,'r').readlines()
        for annotation in annotations:
            annotation_list = annotation.split(' ')
            wavename = Path(annotation_list[0]).name+'.wav'
            words = annotation_list[1:]
            words_str = ' '.join(words)
            print("Annotation of wavefile {} is: {}".format(wavename,words_str))
            ipa = check_output(["espeak", "-q", "--ipa", "-v", "en-us", words_str]).decode("utf-8")
            print("IPA translation: {}".format(ipa))
            cmd = '''INSERT INTO {} VALUES (?,?,?,?,?)'''.format(tablename)
            c.execute(cmd, (session,wavename,words_str,ipa,language))
            conn.commit()
            print("Annotation saved successfully.")
    return None
```
Note: I stuck with using 'American English' for IPA translation as Voxforge <a href="http://www.voxforge.org/home/forums/message-boards/acoustic-model-discussions/using-voxforge-acoustic-models-with-british-english-and-other-accents/6">apparently uses the CMU dictionary of American English</a> for its English speech database.

How this code puts annotations and their IPA translations into a database looks like this:

### View of Sqliteman table
![Imgur](https://i.imgur.com/gzrbKJF.png?1)
##### The path is saved in 'session_ID', which contains the langauge label and recording session name; the wavefile name of each annotation is saved under 'filename'. Each recording session of a speaker contained several wavefiles.

Anyways, the above code works for the files if they are already extracted; unfortunately, I don't have enough space on my computer to extract all of the zipfiles and then get the annotation information. Therefore, similarly to how I extracted the MFCC data from the wavefiles, I will have to get the annotation information as I unzip each zipfile.

The code for that looks like this:

First reworking the definitions to apply to many tgz files:

```
import os, tarfile
import glob
from pathlib import Path
from subprocess import check_output

def annotations2IPA(c,conn,tablename,path,language,session):
    annotation_file = path + 'etc/PROMPTS'
    annotations = open(annotation_file,'r').readlines()
    for annotation in annotations:
        annotation_list = annotation.split(' ')
        wavename = Path(annotation_list[0]).name+'.wav'
        words = annotation_list[1:]
        words_str = ' '.join(words)
        print("Annotation of wavefile {} is: {}".format(wavename,words_str))
        ipa = check_output(["espeak", "-q", "--ipa", "-v", "en-us", words_str]).decode("utf-8")
        print("IPA translation: {}".format(ipa))
        cmd = '''INSERT INTO {} VALUES (?,?,?,?,?)'''.format(tablename)
        c.execute(cmd, (session,wavename,words_str,ipa,language))
        conn.commit()
        print("Annotation saved successfully.")
    return None

def collect_tgzfiles():
    tgz_list = []
    #get tgz files even if they are several sudirectories deep - recursive=True
    for tgz in glob.glob('**/*.tgz',recursive=True):
        tgz_list.append(tgz)
    return tgz_list

def extract(tar_url, extract_path='.'):
    tar = tarfile.open(tar_url, 'r')
    for item in tar:
        tar.extract(item, extract_path)
        if item.name.find(".tgz") != -1 or item.name.find(".tar") != -1:
            extract(item.name, "./" + item.name[:item.name.rfind('/')])
    return None

def tgz_2_annotations(tgz_list,c,conn,tablename):
    if len(tgz_list)>0:
        tmp_path = '/tmp/annotations'
        for t in range(len(tgz_list)):
            extract(tgz_list[t],extract_path = tmp_path)
            session_info = Path(tgz_list[t])
            #print("Path: {}".format(session_info.parts))
            language = session_info.parts[0]
            #print("Language label: {}".format(language))
            session_id = os.path.splitext(session_info.parts[1])[0]
            #print("session ID: {}".format(session_id))
            newpath = "{}/{}/".format(tmp_path,session_id)
            annotations2IPA(c,conn,tablename,newpath,language,session_id)
            return True
    return False
```
Implementing those defintions into code:

```
#interacting with paths/folders
import os

#handling, organizing, and saving data
import sqlite3
from sqlite3 import Error

#IPA extraction (international phonetic alphabet)
from collect_speech_data import collect_tgzfiles, tgz_2_annotations


#global variables:
database='speech2IPA4.db'
tablename = 'speech2IPA'


#initialize database
conn = sqlite3.connect(database)
c = conn.cursor()

#create the table (if it isn't there)
c.execute("CREATE TABLE IF NOT EXISTS {}(session_ID TEXT, filename  TEXT, annotation_original TEXT, annotation_IPA TEXT, label TEXT) ".format(tablename))
conn.commit()

#collect all of the tgz files to be extracted
tgz_list = collect_tgzfiles()

#collect annotations and save to database
collected = tgz_2_annotations(tgz_list,c,conn,tablename)
if collected:
    print("Annotations have been collected.")

```



## Ideas:
* Perhaps use <a href="http://espeak.sourceforge.net/languages.html">Espeak</a> to translate text to IPA? This could allow for languag teaching applications: show how to pronounce various words in the relevant language. Communicate with others thru different languages
* Remove silences from speech
* Create pictures of whole- mel-log-spectrograms and train with IPA sequences (do these need to be same length?)
* Can I take into account length of IPA segment and length of speech, divide them accordingly, with windows, and supply the little window of spectrogram as input and output two or three IPA letters, that should correspond to the segment of speech after silence removal?

## Useful resources:
* <a href="http://www.arthaey.com/markdown-ipa.html">IPA to markdown</a>

