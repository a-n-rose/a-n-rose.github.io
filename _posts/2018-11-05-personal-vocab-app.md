---
layout: post
title: "Developing my own vocab collection application"
date: 2018-11-05
---

I'm finally putting it together: my own personal vocabulary etc. collector! High five to self. I have several goals for this project: 

1) create a knowledge collector that works for me

2) improve my overall organization of code and functionality

3) practice unittesting development with SQLite3 and OOP functionality

and eventually

4) increase the complexity of the overall functions: add NLP, speech processing, web-scraping, etc.

All of the above will likely be constant 'works-in-progress', which is good. Otherwise it probably means I'm dead. Or bored of this project. 

Anyways, to be honest, I wanted to develop this vocab app with a mindset of "test-driven development". That lasted maybe 2 hours. I found that I first needed a basic version of the app to figure out a working layout. To that I would create and apply tests. Those plans for developing unittests *did* help me to keep functionality more separate, though, so in a way I did build it in a test-driven way..

The current version can be found in this <a href="https://github.com/a-n-rose/Vocab-Collector-and-Tester">repo</a>.

## Version 1

My first version is pretty simple. There is no GUI; one uses the app via command line. I built it with Python 3.5 so the only installation necessary is Numpy. Otherwise, all other libraries are built in. 

Here is a basic run through once you run 'vocab_run.py'. 

First prompt:

```
Username: 
Spaces and special characters will be removed: 
```
Just for fun I'll enter in  
```
$ $A)#ni__k   a*( 
```
which will get saved as 'Anika' as spaces and special chars get removed from usernamee. This will result in the output:
```
Your username will be saved as 'Anika'

Welcome Anika! Enter a password to create your account

Password: 

```
I'll enter a fancy password, cause I don't want anyone to ever find out the ridiculous words I'm learning :P 

```
You're account has been created.
Get started by creating your first list.

For each list of words you create, you can add tags and example sentences.

You can even test your knowledge with flashcards, multiple choice and fill-in-the-blank quizzes.


Action:
1) open existing list
2) create new list
Enter 1 or 2 (or exit): 

```

Just to see what happens, I'll enter '1':
```
It looks like you don't have a list. Start one now!
Name of list: 
```
I need to brush up on my Spanish so I'll go with 

'Spanish 101'

The next prompt is:
```
Tags (separated by ;)
```
I'll enter 

'brush-up; basic; every-day'

Great! The list has been created. My next menu is the word menu:
```
Action:
1) add word
2) review words 
3) change list
Enter 1, 2, 3 (or exit): 
```

Well, I gotta add some words.

```
New word: 
```
hola
```
Meaning: 
```
hello
```
Example sentence (if multiple, separate by ; ) 
```
Hola, ¿cómo estás?
```
Tags (separated by ;)
```
greeting


Once I've added that word, I get the same menu again:
```
Action:
1) add word
2) review words 
3) change list
Enter 1, 2, 3 (or exit): 
```

This time I'll show you some 'review' or quizzing functionality by entering:

2


```
Review:
1) Flashcards
2) Multiple Choice
3) Fill in the blank
4) View all words and their meanings
Enter 1, 2, 3, 4 (or exit): 
```
I want to show you 'fill-in-the-blank'

3

```
Enter the word that fills the blank.


____, ¿cómo estás?
Your answer:
```
Hola
```

Way to go!


Your score: 100.0% 
Perfect score! I think you gotta find some harder words.

Action:
1) add word
2) review words 
3) change list

```
## What I'm working on now

I'm currently bouncing back and forth between *creating unittests* and improving/*building onto functionality*. For example, I want to keep track of users' performances on their lists, to be able to quiz them if they haven't tested words in a while, as well as so many other things. 
