---
layout: post
title: "Building a ConvNet+LSTM neural network to classify female vs male speech: A Python workshop for PyLadies programmers"
date: 2019-01-13
---


## Workshop Goal

I want people from all backgrounds to be able to learn something from this workshop (<a href="https://github.com/a-n-rose/workshops/tree/master/deep_learning_acoustics">workshop repository</a>). Before we delve in, decide for yourself what is most important for you to learn? What skills do you want to walk away with?

* How to work with speech? 

* New Python tricks? 

* How to set up and deploy neural networks? 

* Learn how to reimplement or further explore research findings? 

We have a **lot** to cover so try to keep focused on what is most relevant to you. I have set up this workshop to allow people from all levels to walk away with something, if nothing else than motivation to start your own Python projects. (If you're interested in peaking through my own array of projects, have a look <a href-"https://a-n-rose.github.io/projects/">here</a>.) 

## Background 

I will only cover the basics of what is used in current research and what I think is the most useful for you to know. (For a very professional implementation of neural networks and sound, with some nice object oriented programming and module organization, check out this <a href="https://github.com/locuslab/TCN/tree/master/TCN/poly_music">beautiful repo</a>. I'm just not that far yet..) All of what I cover is free and open source.

According to <a href="https://www.topbots.com/">TopBots</a>,"the largest publication, community, and educational resource for business leaders applying AI to their enterprises" (according to TopBots), the ten most important artificial intelligence (AI) <a href="https://www.topbots.com/most-important-ai-research-papers-2018/#ai-paper-2018-10">papers</a> completed in 2018 included one comparing convolutional neural networks (CNN or ConvNet) and recurrent neural networks (RNN). They found that, even with time series data, such as language, CNNs create better models than those built using RNNs (under which long short-term memory (LSTM) neural networks are categorized). 

    “Always start with a CNN before reaching for an RNN. You’ll be surprised with how far you can get.” – Andrej Karpathy, Director of AI at Tesla.
    
Important to note here, is that none of those very important papers specifically targeted speech. We will explore in this workshop how ConvNets and LSTMs (a subset of RNNs) handle speech data. We will build a ConvNet, an LSTM, and a ConvNet+LSTM and see how they compare when handling real-world data. 

If you're asking yourself, WTF is a CNN, RNN, or LSTM? No worries, I'll cover the basics. 

## Neural Networks

What I find the most helpful when thinking about neural networks is the *kind* of data they work with. 

* Object Identification (e.g. <a href="https://towardsdatascience.com/mtcnn-face-detection-cdcb20448ce0">face detection</a>, ): Images

* Text Analysis (i.e. classifying positive or negative reviews): Series

* 

## Speech

## Python


