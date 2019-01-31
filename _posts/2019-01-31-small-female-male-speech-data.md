---
layout: post
title: "Download Female and Male speech Saarbrücker Voice Database"
date: 2019-01-31
---

## Directions for downloading Female and Male speech data

This is a pretty small dataset but potentially useful. Data is available from Saarbrücker Voice Database. Explore feature extraction and convolutional neural network (CNN), long short-term memory networks (LSTM), and CNN+LSTM in this <a href="https://github.com/a-n-rose/speech-recognition-projects/tree/master/speech_commands">repo</a>.

Request the data from here: 

<a href="http://www.stimmdatenbank.coli.uni-saarland.de/index.php4#target">Saarbrücker Voice Database</a>

## Male Speech Data

I've posted some screenshots to help you navigate the webpage. We will first download healthy male speech, then healthy female speech. The webpage is in German, so real quick: 

healthy = gesund

male = männlich

female = weiblich


1) Check the boxes for "männlich" and "gesund"

2) Click "Exportieren" at the bottom of the page.

![Imgur](https://i.imgur.com/KcKSA4x.png)

3) Check the boxes "Satzdatei" and "WAV" for the 'Sprach-Signal' (not the 'EEG-Signal').

4) Once the data has loaded, click on "Herunterladen". Save this zip file in the workshop's "data" folder as "male_speech.zip". Extract the zipfile. 

![Imgur](https://i.imgur.com/26Fz5EE.png)

5) Rename the extracted file "export" as "male_speech"

![Imgur](https://i.imgur.com/SHhfjxM.png)
##### This is just to show how the export file should be renamed to "male_speech" and "female_speech"

6) On the database website: click on "Zurück" and "Neue Anfrage"

This should reset the values and you can do the same to download the female speech.

### Female Speech Data

Now, repeat the process for the female speech: 

1) Check the boxes for "weiblich" and "gesund"

2) Click "Exportieren"

3) Check the boxes "Satzdatei" and "WAV" for the 'Sprach-Signal' (not the 'EEG-Signal').

4) Once the data has loaded, click on "Herunterladen". Save this zip file in the "data" folder as "female_speech.zip". Extract the zipfile. 

5) Rename the extracted file "export" as "female_speech"

### Check the speech

Ensure the downloads were successful. You should have 252 wave files of men and 382 wave files of women saying the phrase "Guten Morgen, wie geht es Ihnen". 
