---
layout: post
title: "Save to SQL with Python: CPU and Time with For Loops vs Panda/Lambda"
date: 2018-11-06
---

While putting together my new-and-improved <a href="/2018/11/05/updated-babyname-recommender.html">babyname recommender</a>, I saw the opportunity to further explore the advantages and disadvantages in techniques for saving data to a database. 

In my scenario, I am working with an SQL database to insert and save the names, sexes, and year popularity of all names from USA social security applications, ranging from the years 1880 until 2017. (The name data can be downloaded from <a href="https://catalog.data.gov/dataset/baby-names-from-social-security-card-applications-national-level-data">data.org</a>.)

I will compare the times and CPU usage of the various techniques I employ.

## My computer specs

<a href="https://www.cnet.com/products/acer-aspire-r-14-r3-471t-54t1-14-core-i5-4210u-6-gb-ram-1-tb-hdd/specs/">Acer Aspire R3-471 series</a>

My computer is dual core, with quite a bit of memory. It has a poor graphics card, though. I would say it is a pretty mediocre computer. 

Your machine strengths and weaknesses can help you determine the best way to handle data, especially larger amounts of it. 

## Name Data

Let's get a quick look at the data. 

### Number of Unique Baby Names in the USA
![Imgur](https://i.imgur.com/BpF9BdM.png)
#### This graph shows the number of unique names give to babies each year. Note: all names included were used in their corresponding year at least 5 times, with some almost 10,000 times (some years had around 10,000 US American babies all with the same name). 

There is a lot more name data as the years progress. I imagine much of this has to do with the increase in population. But I also wonder if it is has cultural correlations? For example, perhaps in more modern times, parents wanted more unique names for their children. Therefore, more name data results also because more babies have different names.

### Number of Babies per Year (based on US social security applications)
![Imgur](https://i.imgur.com/rIzca6f.png)
#### This graph shows the total number of babies recorded that year (with a name popularity of at least 5, meaning at least 5 babies must have been given that name)

This graph looks very similar to Figure 1, which means that population probably accounts for the increase in baby name data through the years. But it is still interesting to see if unique names increase, decrease, or stay the same through the years.

### Percentage of Babies with Most Popular Name
![Imgur](https://i.imgur.com/kbgoxXs.png)
#### The percent of babies with the year's most popular name goes down as years progress. This could mean that parents are trying to give their babies more unique names in more recent times. 

This helps give us an idea of how the data looks. As the years progress, more name data will have to be inserted into the database, a relavant issue to think about when writing code for that. 


## For Loop and SQLite3

The first technique I implemented did a pretty good job not taking over my CPU. When I ran it, my CPU mostly remained at around 20% - 25%, with the rare occasion going above 30%. This meant that I could use my computer as usual, while my program collected, organized, and safely saved my data for me. 

The time to go through all the data, however, took forever. I started the program 3 hours ago and it is only 36% through the entire dataset. Further, as there is more baby name data to work with as the years progress, this program took longer and longer to process each year.

As I continue working with this data, I will update this post. 
