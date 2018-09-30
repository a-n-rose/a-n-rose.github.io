---
layout: post
title: "Training an LSTM on non-aligned annotation and speech data"
date: 2018-09-28
---

Speech data available for model development usually fits in one of these two categories: 1) little but meticulously annotated speech (and probably not publicly available) and 2) **a lot** but largely unlabeled speech data (maybe publicly available). My experiment below explores the use of <a href="http://voxforge.org/">Voxforge's</a> publicly available and free speech data, including their simple annotations of that speech.

Voxforge hosts several large, publicly available speech databases for multiple languages. Included in their speech databases are annotation files associated with individual recordings. I had already downloaded several speech databases in aims of exploring how to <a href = "https://a-n-rose.github.io/2018/08/22/language-classifier.html">train neural networks</a> on <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">MFCC data</a>. Therefore, I decided to see how I could apply the annotations to the training data for an LSTM neural network. Could I build a speech to text model?

Generally speaking, I would like to see how speech recognition models can be built without aligned annotations, meaning annotations that do not exactly correspond with the timing of speech sounds. There are models built on either relatively small amounts of professionally annotated speech data and also on large amounts (i.e. big data) of unannotated speech data. Both have their role in model development and can be used together to build more complex and telling models (<a href="https://www.icsi.berkeley.edu/icsi/blog/data-versus-experts">this blog</a> explores this topic in regards to text analysis; this paper by <a href="https://www.semanticscholar.org/paper/Speech-Analysis-in-the-Big-Data-Era-Schuller/3c9567ef956e5ea1a0a20843aea09489d117f5ac">Schuller, 2015</a> reviews data available today, and ways to best analyze them.) 

I would like to know if speech data with approximate annotations offer any sort of contribution to the analysis of speech. If so, this may improve the effectiveness of some speech data, even though the speech data is not professionally annotated. This was the purpose of my experiment with Voxforge's English speech database. While I would like to build multilinguistic models, I explored first if I could develop working models with one language. Click <a href="https://github.com/a-n-rose/language-classifier/tree/master/english_speech_to_ipa">here</a> for the repository.

## Step one: collect and store the data

The first step in collecting the data I needed was to understand the <a href="/2018/09/12/multiling-speech2text.html">architecture</a> of the database(s), specifically where the annotations and corresponding wave files were stored. Once I determined a pattern, I wrote a script that unzipped each zip-file individually, extracted the data I needed, and then deleted the unzipped contents. I stored the annotations and the MFCC data in the same database but in different tables. I used the zipfile and wave file names as keys to match the right annotation with the right set of <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">MFCC</a> values.

The data I extracted from these files, which I will explain further below, included 1) international phonetic alphabet (<a href="https://en.wikipedia.org/wiki/International_Phonetic_Alphabet">IPA</a>) representations of the annotations and 2) MFCC data representing the audio data.

As I extracted the written annotation of each recording, I translated the annotation into characters of the IPA using Espeak, <a href="http://espeak.sourceforge.net/">an open source speech synthesizer software</a>. The reason I wanted the transcripts in the IPA was because the IPA was developed to represent all sounds in langauges. In a language's alphabet, often times a single sound is represented by several letters. For example, in German, *sch* - pronounced *sh* as in *shoe* - and *dsch* - pronounced *j* as in *jungle* - are sounds with a lot of letters that can be represented in IPA with fewer characters: ʃ  and dʒ, respectively.

I also extracted <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">MFCC</a> data from each recording. While extracting, I applied my <a href="https://a-n-rose.github.io/2018/09/06/updating-VAD.html">voice activity detection (VAD) function</a> to avoid collecting MFCC data during the beginning and ending silences of recordings. Note: I extracted the MFCC data twice, once with added noise, as I found that <a href="/2018/08/22/language-classifier.html">aided model generalizability</a>, and once without added noise, mainly for comparison. 

### Table 1: Filenames, annotation, IPA 'translation' information
![Imgur](https://i.imgur.com/gzrbKJF.png?2)

### Table 2: MFCC values (columns 0 - 39) with additional necessary information.
![Imgur](https://i.imgur.com/7dVuL6n.png?1)

The data were collected with Python scripts and saved in a SQL database. The purpose of this experiment was to serve as a test round. Therefore, I only collected a total of 100 recordings, which worked out to be appx. 5 minutes of speech data. For code relevant for these tasks, please see <a href="https://github.com/a-n-rose/language-classifier/blob/master/english_speech_to_ipa/collect_speechdata.py">'collect_speechdata.py'</a> for the functions, <a href="https://github.com/a-n-rose/language-classifier/blob/master/english_speech_to_ipa/sql_data.py">'sql_data.py'</a> for how I saved/loaded data, and <a href="https://github.com/a-n-rose/language-classifier/blob/master/english_speech_to_ipa/collect_IPA_MFCC_English_tgz.py"> 'collect_IPA_MFCC_English_tgz.py'</a> for the main module.

## Step two: combine data/prep data for training models

The IPA and MFCC data needed to be paired so the trained model could classify the MFCC data as certain IPA characters. Given my experience with <a href="/2018/09/07/language-classifier-LSTM.html">both simple artificial (ANN) and long short term memory (LSTM) neural networks</a>, I knew I wanted to feed the data to an LSTM network. Therefore, I prepared the data so that I could feed it to the network in sequences. The model would likely learn IPA characters if it trained on MFCC data in short sequences which would capture small speech sounds. For further clarification on this topic, please refer to this <a href="http://practicalcryptography.com/miscellaneous/machine-learning/guide-mel-frequency-cepstral-coefficients-mfccs/">post</a>. 

Furthermore, for good practice, I randomly assigned each recording sample/annotation to either the training, validation, or testing datasets, with a ratio of 6-2-2, respectively.

In order to align the IPA and MFCC data for each recording, I calculated the length of the IPA characters, which should have better represented length of the utterance than the original alphabet would. I then calculated the number of MFCC rows linked to the annotation and divided that number by the number of IPA characters. This would serve as a guideline for how many MFCC sequences would represent each IPA character. Note: one dataset included only the IPA letters (e.g. 's','i' etc.) and another dataset IPA letters as well as some stress markers (e.g. 1) ˌ 2) ˈ 3) ː , which designate secondary stress, primary stress, and long vowel length, respectively). I wanted to compare which worked better classifying speech sounds as IPA characters.

Lastly, I had to decide how many MFCCs I would use to train the network to classify IPA characters. I decided that classifying three IPA characters would be ideal, as when speech sounds are produced, they are not produced in isolation: their sounds are influenced by the sounds produced before and after. (To see for yourself, see how the letter *j* (dʒ in IPA) changes when you say *joke* (dʒəʊk) versus *judge* (dʒʌdʒ). Your lips are already getting round while saying *j* when pronouncing *joke* while your lips stay more relaxed with the latter.) This meant that I would feed the LSTM network with samples in sequences of MFCCs that corresponded to three IPA characters. Not every speaker speaks at the same speed; therefore sometimes more MFCCs were necessary to represent IPA characters and sometimes fewer. On average 6 MFCC samples represented three IPA characters, so I limited the MFCC sequences to 20, and zero padded the sequences (what was most) that were less than that. 

Lastly, I had to turn the label data (i.e. three IPA characters) from strings into integers, data the LSTM could work with. To do so, I generated a list of all 3-letter IPA combinations possible, including letter repeats. I then applied the indexes of these as labels of the MFCC sequence data. This way I could avoid using one-hot-encoding (I tried that and got a memory error), but I'll get into that later. 

### Table 3: MFCC data with IPA label
![Imgur](https://i.imgur.com/4vvSciZ.png?1)
#### First column (not visualized) is the dataset assignment (train, validate, test); the next 40 columns pertain to the 40 MFCC coefficients; the last column on the right pertains to the IPA label. The first set of MFCC data needs 15 samples to identify 3 IPA characters. In order to make sure each set of samples had the same lengths, any samples less than 20 were padded with zeros. You can see the next sample following the zeros had a different 3 letter IPA label.

I prepared the scripts to allow for adjustment of two key variables: 1) the IPA window shift and 2) which IPA characters were excluded from analysis. I built a total of eight models, trained on data with a mix of the following options:

1) with or without added noise to the MFCC data

2) with or without IPA stress symbols: e.g. ˌ  ˈ  ː 

3) with or without overlap 

To give you an idea of what the window shift/ overlap variable looks like, let's take the word *shallow* as an example. Here is what the '3-letter labels' would look like if they had overlap:

* sha, hal, all, llo, low

Without overlap:

* sha, llo 

Every label required a full 3-letter representation; if three letters were not avaiable at the end of the utterance, those letters/MFCC data were not included in the training (hence the disappearance of *w* in the non-overlap variable example above).

To see the code I wrote to prepare the data, please refer to <a href="https://github.com/a-n-rose/language-classifier/blob/master/english_speech_to_ipa/batch_prep.py">'batch_prep.py'</a> for the functions and <a href="https://github.com/a-n-rose/language-classifier/blob/master/english_speech_to_ipa/combine_align_ipa_mfcc_data.py">'combine_align_ipa_mfcc_data.py'</a> for the main module.

## Step three: build and train the model

Even though I only used 5 minutes of speech data, that worked out to be 26,280 samples of MFCC data for the condition without overlap and 76,640 samples of MFCC data for the condition with overlap. I could not feed the data all at once to the LSTM model and instead developed a generator function that fed the network 20 sequences of MFCC data at a time. To see the code relevant for the building of the LSTM, please refer to the script <a href="https://github.com/a-n-rose/language-classifier/blob/master/english_speech_to_ipa/batch2model.py">'batch2model.py'</a>, which imports the 'sql_data.py' and 'batch_prep.py' scripts mentioned earlier.

At this stage of experimentation, I used only training and test datasets. If I continue experimenting with this type of data I would also include a validation dataset. 

I found most staggering differences in accuracy rates between the models that use the stress characters and those that didn't. Otherwise, the accuracy rates remained at approximately the same levels. 

### Table 4: LSTM Model Performance
![Imgur](https://i.imgur.com/ntkF8uL.jpg)
#### Model types: A = trained without overlapping MFCC data and MFCC data *with* added noise; B = trained *with* overlapping MFCC data and *with* added noise; C = trained without overlapping MFCC data and without added noise; D = trained *with* overlapping MFCC data and without added noise. 

Clearly 27-28% accuracy does not look very high; however, when considering the number of labels the models had with which to classify the MFCC samples (704,881 labels to be exact), that percentage looks a lot more impressive. It is interesting to compare the difference between the accuracy of those models, which were trained on MFCC data that was aligned with IPA characters *with* stress markers, and the accuracy of the models that were aligned with IPA characters *only*: the stress markers were not included in the alignment stage. To better understand what this accuracy means, I would like to apply the models to brand new data and see how the models classify the speech. 

I would be curious to further explore training models with these loosely aligned speech and annotation data; however, the computation costs of such development is high. To apply this technique to all of the available English data on Voxforge, the MFCC extraction alone would take at least 9 hours. The following steps of MFCC and IPA alignment and then the training of the LSTM model, several days would likely be needed. In addition to that, memory costs would also be very high. 

If I did further explore a model, I would likely explore a model trained on MFCC data *with* noise added and aligned with IPA characters *with* stress markers. In <a href="/2018/08/22/language-classifier.html">past experiences with neural networks</a>, I found the models trained on data with added noise generalized better to realworld data. Also, even though there was a slight increase in accuracy for the models trained with overlapping MFCC data, that increase is not large enough to make up for the more than double amount of data necessary.

In sum, the findings of this small experiment reveal that IPA characters - when used with their <a href="https://en.wikipedia.org/wiki/Suprasegmentals">suprasegmental</a> markers - can be used to create loose alignments with speech productions and result in potentially reliable training data. The findings also reveal the reliability of Espeak's software which translated the English text to IPA letters. My next step is developing an application to collect new English speech and apply these models to see which handles realworld data best.

### Resources:

* Blog on NLP model development with <a href="https://www.icsi.berkeley.edu/icsi/blog/data-versus-experts">small data with annotations vs big data</a>

* Schuller, B.W. (2015). Speech Analysis in the Big Data Era. TSD.

* Very helpful <a href="http://adventuresinmachinelearning.com/keras-lstm-tutorial/">tutorial</a> on using Keras to build LSTM neural networks.
