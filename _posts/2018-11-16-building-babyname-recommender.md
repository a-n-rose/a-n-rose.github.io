---
layout: post
title: "Babyname Recommender: Building a Recommender in Python Using KMeans Clustering"
date: 2018-11-16
---

I put together a basic version of my <a href="https://github.com/a-n-rose/recommendation-systems-python/tree/master/babyname_recommender">'Babyname Recommender'</a>. In this first version, the user has the options to create searches, rate names, and to get name recommendations. They can create an (almost) infinite number of searches, in which the corresponding name ratings will be saved. This means that they can create searches for only girl names, only boy names, or all names, for their children, pets, or random items they want to name (with real human names). The names they like or don't like will only get saved in that particular search. The user will also get a list of recommended names based on the names they like for each search. 


## Bulding the Recommender

Before a recommender has a user's data, it needs somewhere to start. One way to start is to define clusters of the items the recommender is to recommend: if a user likes items from certain clusters, the recommender can recommend other items from those clusters. 

As the user rates more items, how the clusters are defined can be adjusted.

### Creating Clusters

To group objects, such as names, into clusters, we need to decide how we would group them: by length? gender? letters? phonology? background? etc. 

To start out, I will group them in basic clusters using the data I have already: letters. I will use sklearn and K-means clustering to help me out. 

First, I need to get the name data into a form an algorithm such as K-means clustering would understand. To do this, I created a new table with each letter in the English alphabet as its own column (in alphabetical order) and length of the name as its own column. 

With this kind of table, I can put in binary or integer values for each name, whether or not they have this letter and how long the name is. (Note: later I will improve these features, for example, scaling the length, and expanding on the sounds in the name, to show how that influences the performance of the recommender)

### Table 1: Letter and Length Feature Columns
![Imgur](https://i.imgur.com/Fdmqv3m.png?1)
#### Note: see Table 2 to check if the names are labeled correctly. For example, row 1 represents 'Mary'. That row should have all zeros except for columns 'a', 'm', 'r', and 'y', as well as a number 4 in the 'length' column.

### Table 2: Reference Table: Contains Name Id
![Imgur](https://i.imgur.com/5rW7yHP.png)
#### Above: the first 13 names in the database.

When fitting the Kmeans algorithm to the data, how many clusters are used is very important.

I was not sure how many clusters would be useful for this name recommender. Therefore, I created a table that collects the clusters assigned to each name (based on its features) with 9, 30, and 50 clusters. Note: Sklearn's Kmeans module has a default of 8.

The reason I chose 9 to start out with is the following:

The names from the database range from lengths of 2 to 15. I personally would create five categories for these lengths: very-short, short, middle, long, very-long

Considering the letters, 26 of them, we've got vowels and consonants to think about: 5 vowels (sometimes six), and usually 21 consonants. Simply put, vowels tend to be more open or closed (e.g. *a* in *and* vs *e* in *end*) and consonants voiced or unvoiced (e.g. *p* in *pun* vs *b* in *bun*). 

In sum, 5 clusters for length and 4 clusters for speech sounds in letters, which brings us to 9. Ultimately I didn't expect this to work but since I had no idea where to start, this is where I did. It was after experimenting a bit that I was able to tell that 9 clusters were probably too few, which inspired me to try 30 and 50 as well. 

### Testing the Clusters

First, I measured how much time it took to assign the clusters to all of the names (107,973 names to be exact).

| Num Clusters:  | Duration:      | 
| --- | --- |
| 9 | 9.69 sec |
| 30 | 44.61 sec |
| 50 | 122.78 sec  |

To see how each of the cluster assignments compared, I ran the recommender with names I had rated, one search for a girl name and one search for a boy name. I tested the cluster assignments as I rated. 

To do so, I looked at the clusters assigned to the names I liked and looked to see how many of those clusters were also in the clusters assigned to the names I didn't like, i.e. I measured the similarity between the clusters of my liked names and the clusters of my disliked names. If there were quite similar, the clusters weren't being very useful.

How I measured similarity:

```
def check_cluster_list_similarity(clusterlist1,clusterlist2):
    total_len_cluster1 = len(clusterlist1)
    sim = 0
    for cluster in clusterlist1:
        if cluster in clusterlist2:
            sim+=1
    if total_len_cluster1 > 0:
        similarity = sim/float(total_len_cluster1)
        return similarity
    return None
```


## 9 Clusters

| Search:    | Num Names Rated:   | Cluster Similarity:   |
| :--------- |:---------:        | :------------------------------:  |
| Girl Name |  12               |                71.43%             |
| Boy Name  |  67               |                100%               |
| Girl Name | 115               |                98.04%             |
| Boy Name  | 206               |                100%               |

## 30 Clusters

| Search:    | Num Names Rated:   | Cluster Similarity:   |
| :--------- |:---------:        | :------------------------------:  |
| Girl Name |  12               |                28.57%             |
| Boy Name  |  67               |                92.0%              |
| Girl Name | 115               |                98.04%             |
| Boy Name  | 206               |                100%               |

## 50 Clusters

| Search:    | Num Names Rated:   | Cluster Similarity   |
| :--------- |:---------:        | :------------------------------:  |
| Girl Name |  12               |                14.29%             |
| Boy Name  |  67               |                72.0%              |
| Girl Name | 115               |                86.27%             |
| Boy Name  | 206               |                100%               |


As you can see, the more names I rated (I liked around 40% of the names presented to me), the more and more similar the liked and disliked clusters became. 

Let's see how adjusting these features helps!

