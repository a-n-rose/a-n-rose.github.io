---
layout: post
title: "PyLadies Workshop: Building a ConvNet+LSTM neural network to classify female vs male speech"
date: 2019-01-13
---


## Workshop Goal

I want people from all backgrounds to be able to learn something from this workshop (<a href="https://github.com/a-n-rose/workshops/tree/master/deep_learning_acoustics">workshop repository</a>). Before we delve in, decide for yourself what is most important for you to learn? What skills do you want to walk away with?

* How to work with speech? 

* New Python tricks? 

* How to set up and deploy neural networks? 

* Learn how to reimplement or further explore research findings? 

We have a **lot** to cover so try to keep focused on what is most relevant to you. I have set up this workshop to allow people from all levels to walk away with something, if nothing else than motivation to start your own Python projects. (If you're interested in peaking through my own array of projects, have a look <a href="https://a-n-rose.github.io/projects/">here</a>.) 

## Background 

I will only cover the basics of what is used in current research and what I think is the most useful for you to know. (For a very professional implementation of neural networks and sound, with some nice object oriented programming and module organization, check out this <a href="https://github.com/locuslab/TCN/tree/master/TCN/poly_music">beautiful repo</a>. I'm just not that far yet..) All of what I cover is free and open source.

According to <a href="https://www.topbots.com/">TopBots</a>, "the largest publication, community, and educational resource for business leaders applying AI to their enterprises" (according to TopBots), the ten most important artificial intelligence (AI) <a href="https://www.topbots.com/most-important-ai-research-papers-2018/#ai-paper-2018-4">papers</a> completed in 2018 included one comparing convolutional neural networks (CNN or ConvNet) and recurrent neural networks (RNN). They found that, even with sequence data, such as language, CNNs create better models than those built using RNNs (under which long short-term memory (LSTM) neural networks are categorized). 

# “Always start with a CNN before reaching for an RNN. You’ll be surprised with how far you can get.” – Andrej Karpathy, Director of AI at Tesla.
    
Important to note here, is that none of those very important papers specifically targeted speech. We will explore in this workshop how ConvNets and LSTMs (a subset of RNNs) handle speech data. We will build a ConvNet, an LSTM, and a ConvNet+LSTM and see how they compare when handling real-world speech data. 

If you're asking yourself, WTF is a CNN, RNN, or LSTM? No worries, I'll cover the basics. 

## Neural Networks

What I find the most helpful when thinking about neural networks is the *kind* of data they work with. 

### Convolutional Neural Networks

ConvNets are most well known for their application to image analysis. These can be used to create <a href="https://towardsdatascience.com/mtcnn-face-detection-cdcb20448ce0">face detection</a> models, object identification models (<a href="https://towardsdatascience.com/object-detection-with-neural-networks-a4e2c46b4491">tutorial</a>), written digit classifiers (<a href="https://machinelearningmastery.com/handwritten-digit-recognition-using-convolutional-neural-networks-python-keras/>tutorial</a> using MNIST dataset"), and others.

### Long Short-Term Memory Neural Networks

LSTMs are a subset of recurrent neural networks (<a href="https://towardsdatascience.com/recurrent-neural-networks-and-lstm-4b601dd822a5">RNN</a>). They are known for being able to train with data that requires more flexible memory. LSTMs are capable of remembering and forgetting information that was presented to the algorithm several samples previous. 

## Speech

## Python

## Other Related Tutorials 

* Building a <a href="https://medium.com/analytics-vidhya/building-a-speaker-identification-system-from-scratch-with-deep-learning-f4c4aa558a56">Speaker Identification System</a>.

* Review of machine and deep learning techniques to do <a href="https://nlpforhackers.io/deep-learning-introduction/">sentiment analysis</a> of text.

* Use LSTM on <a href="https://towardsdatascience.com/a-beginners-guide-on-sentiment-analysis-with-rnn-9e100627c02e">sentiment analysis</a> text data.

* Time Series <a href="https://machinelearningmastery.com/time-series-forecasting-supervised-learning/">Crashcourse</a>
