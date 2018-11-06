---
layout: post
title: "Babyname Recommender: Setting up the Database with SQLite3"
date: 2018-11-05
---

Don't get any ideas; I'm not pregnant nor am I planning to be. This is just a really fun project that I built in the past (while I was pregnant). 

Back then I had the goal to understand frontend better, and I implemented a recommender with frontend in mind. I learned a lot but I'm happy to further explore this recommendation system in just the backend world.

The repo and code can be viewed <a href="https://github.com/a-n-rose/recommendation-systems-python/tree/master/babyname_recommender">here</a>.

I compare different techniques for saving this name data to SQL databases in this <a href="/2018/11/06/compare-ways-to-save-data.html">post</a>. In it I show the time difference in saving the data iteratively via for-loop vs batch insert with the help of dictionaries. Dictionaries are definitely worth your while. 

## Collect the names

The most updated name list can be found at <a href="https://catalog.data.gov/dataset/baby-names-from-social-security-card-applications-national-level-data">data.org</a>. It is a collection of names from USA social security applications, dating from 1879/1880 until, as of writing this, 2017. 

I unzipped the file into a subdirectory of my current working directory 'names'. (Note: the size of the unzipped contents is appx. 24 MiB.)

## Build a Database

This sounds easy but it does require some extra thought. When creating a database, it is easier in the long-run to build a database with sufficient feature columns as well as tables (and there can be many) than to build a database with insufficient columns and tables, only to have to add/change them later. I at least don't have fun doing that, so I try to avoid that. 

For my name database, I need to think about the possible functionality. For example, what I did last time (basically to avoid dealing with this issue) was, I retained only the most recent use of a name. That means that I didn't keep how popular the name 'Anna', for example, was in the year 1903. I can imagine someone (myself included) being interested in exploring names popular in a specific year, and if I store data that way, I can't do that. 

Ideas for functionality:
* check for overall popularity: all-time favorites
* check for names that used to be popular
* organize names by their sounds
* get suggestions based on another name (i.e. sibling names or last name) --> either avoid similar sounds or search for similar sounds

### Explore the name data

The zip file contains several text files, titled like so: 'yob1880.txt'. 'yob' stands for 'year of birth'. Each text file is dedicated to one year, and they spand from 1880-2017. 

Each text file starts first with girl names and ends with boy names. The order of the names corresponds to their popularity/rank (tied names are put in alphabetical order). 

The columns go as follows: 1) name, 2) sex, 3) popularity

Here are the first 6 lines from 'yob1880.txt'
```
Mary,F,7065
Anna,F,2604
Emma,F,2003
Elizabeth,F,1939
Minnie,F,1746
Margaret,F,1578
```

So what does the data look like?

### Figure 1: Number of Unique Baby Names in the USA
![Imgur](https://i.imgur.com/BpF9BdM.png)
#### This graph shows the number of unique names given to babies each year. Note: all names included were used in their corresponding year at least 5 times, with some almost 10,000 times (some years had around 10,000 US American babies all with the same name). 

There are more name data as the years progress. I imagine much of this has to do with the increase in population. But I also wonder if it also has cultural correlations? For example, perhaps in more modern times, parents want more unique names for their children. Therefore, there are more name data as years progress because more babies have more unique names.

### Figure 2: Number of Babies per Year (based on US social security applications)
![Imgur](https://i.imgur.com/rIzca6f.png)
#### This graph shows the total number of babies recorded that year (with a name popularity of at least 5, meaning at least 5 babies must have been given that name)

This graph looks very similar to Figure 1, which means that population probably accounts for most of the increase in baby name data through the years. But it is still interesting to see if unique names increase, decrease, or stay the same through the years.

### Figure 3: Percentage of Babies with the Most Popular Name
![Imgur](https://i.imgur.com/kbgoxXs.png)
#### The percent of babies with the year's most popular name goes down as years progress. This could mean that parents are trying to give their babies more unique names in more recent times. It could also be a sign of the increasing cultural diversity in the US, resulting in much more diverse names.

This helps give us an primitive idea of how the data looks and how it behaves.

### Plan the database

Looking at this and knowing I want to keep as much information in the database as possible I will build a database with the following structure:

1) table with all names and assigned sex 

columns --> name_id; name; sex; 

2) table for all years and the popularity of each name in that year 

columns --> year_id; year; popularity; name_id (links to the name/parent table)

I can add additional tables later, for example tables with phonological information or name meanings (e.g. dream, protector) and backgrounds (e.g. Arabic, Irish).

The reason I separate the data into 2 tables is because this allows the least amount of redundant information to be saved. In one table I can save the name and the sex, without repeats. I don't need to have 200,000,000 rows dedicated to the name Charles. I can have Charles saved safely there, and linked to other data in various years with its ID. What allows me to remove all repeats of the names is the fact I made a separate table for the year and popularity data. The year and popularity data can be linked to the associated name via name ID. 

## Write the Code: General Tips

1) **Appropriately close the database**

One key thing to remember when setting up code to save data to a database is to ensure you can appropriately close the database, despite any errors that may come up. For that, common practice in Python is to use the 'try statement'. The 'try statement' includes also 'except' and, optionally, 'finally' statements (see <a href="https://www.w3schools.com/python/python_try_except.asp">here</a>).

Basically it tries some code and if it fails, it can get taken care of/ handled in the 'except' statement (let's say you don't want an error to stop your entire program). And in the 'finally' statement, anything else you want to be completed gets completed. Like, for example, appropriately closing your database so no data gets lost or messed up. 

2) **Simplify functionality**

I'll be the first to admit it: it is fun to squeeze in as much functionality into one function. But it is a deep pimple to squeeze out if anything goes wrong. If you see your code exceeding 5 indendtations, seperate it into separate functions. That way your code is more understandable for others and yourself and you can also more easily apply unittesting if you so please.

3) **If something is running slow, see if dictionaries help**

My first version of these functions did not use dictionaries; it just inserted data iteratively, as it collected it. It would probably still be running if I hadn't stopped it. 

Knowing whether you should use a for-loop, list-comprehension, dictionary, apply.lambda, dataframes, etc. can sometimes be difficult. Memory constraints can be an issue (which is why I initially when with a for-loop iteration of inserting data) meaning something that works really really fast for smaller amounts of data crashes your computer when you work with millions of rows. You get the best idea by just trying stuff out. That way you develop a feeling for when what works best.

## My code:

I separated my functionality into three modules:

1) main module that calls the others into action

2) function module that holds all the necessary functions

3) error module that holds any errors that I raise.

First, the main module, as of this writing called 'database_setup.py'. It calls certain functions from the 'Setup_Name_DB' class, which gets the functionality going. The functions getting called are: __init__(), which gets called when the class gets initialized; create_table_names(); create_table_popularity(); and collect_save_data(). I will go over those functions below. 

```
import sqlite3
import traceback

from data_prep_babyname import Setup_Name_DB

database = 'babynames_2018.db'

if __name__ == '__main__':
    try:
        bn = Setup_Name_DB(database)
        bn.create_table_names()
        bn.create_table_popularity()
        bn.collect_save_data()
    except Exception as e:
        traceback.print_tb(e)
    finally:
        if bn.conn:
            bn.conn.close()
```

This code imports the more complicated, meaty class 'Setup_Name_DB' from the module 'data_prep_babyname.py'.  

It defines a class called 'Setup_Name_DB'. This class is initiated with a database name. It is programmed to search for text files and extract the data contained in them and save to an SQL database.

Before one can save data to an SQL database, you have to 

1) connect and create a database and

2) form the database skeleton.

Below is code where I do that. First I import the necessary libraries.
I also import an error I defined in 'errors.py' called 'MissingNameID'

Whenever Setup_Name_DB gets initiated, it creates/initializes a database. 

To form the skeleton, at least of the tables into which I want to insert data, I create the tables I need. When creating them, I need to specify the column names and the type of data they will hold. The tables I created here are the 'names' table and the 'popularity' table.

The 'names' table has three columns:  name_ID, name, and sex.

The 'popularity' table has four columns: year_ID, year, popularity, name_ID

The 'name_ID' column links the year data to the name.


```
import sqlite3
import numpy as np
import glob
import os
import time
from sqlite3 import Error


class Setup_Name_DB:
    def __init__(self,database):
        self.database = database
        self.conn = sqlite3.connect(database)
        self.c = self.conn.cursor()
        
    def execute_commit_msg(self,msg,t=None,many=False):
        if t is None:
            self.c.execute(msg)
        else:
            if many == False:
                self.c.execute(msg,t)
            else:
                self.c.executemany(msg,t)
        self.conn.commit()
        return None

    def create_table_names(self):
        msg = '''CREATE TABLE IF NOT EXISTS names(name_id integer primary key, name text, sex text)  '''
        self.execute_commit_msg(msg)
        return None
    
    def create_table_popularity(self):
        #link year popularity table to names table
        msg = '''CREATE TABLE IF NOT EXISTS popularity(year_id integer primary key, year int, popularity int, year_name_id int, FOREIGN KEY(year_name_id) REFERENCES names(name_id) ) '''
        self.execute_commit_msg(msg)
        return None
```

The next function that is called in the main module is 'collect_save_data'  It basically calls several other functions (which I will list below this function) to do the work, and returns a value back to the main module, either success or failure, once the data has been processed. The most interesting of these functions, I would say, the most interesting functions are: 'organize_name_data', 'prep_dict_4_SQL', and 'batch_insert_data'. Those shape the data in a way to work with <a href="https://docs.python.org/2/library/sqlite3.html">batch inserting data</a> into SQL. 

```
    def collect_save_data(self):
        data_dict = self.data_2_dict()
        name_data_dict, year_data_dict = self.organize_name_data(data_dict)
        names4sql, years4sql = self.prep_dict_4_SQL(name_data_dict,year_data_dict)
        success = self.batch_insert_data(names4sql,years4sql)
        return success
        
    def data_2_dict(self):
        #collect filenames
        text_files = self.collect_filenames()
        num_years = len(text_files)
        count_years = 0
        year_data_dict = {}
        for text_path in text_files:
            year_start = time.time()
            with open(text_path) as f:
                data = f.read()
            year = self.get_year(text_path)
            print("Processing names in the year {}".format(year))
            name_data =  self.separate_name_data(data)
            year_data_dict[year] = name_data
        return year_data_dict
        
    def collect_filenames(self):
        text_files = []
        for txt in glob.glob('./names/*.txt'):
            text_files.append(txt)
        text_files = sorted(text_files, reverse = False)
        return text_files
        
    def get_year(self,path2file):
        filename = os.path.splitext(path2file)[0]
        year = filename[-4:]
        return year
        
    def separate_name_data(self,data):
        name_data = [s.strip().split(',') for s in data.splitlines()]
        return name_data
    
    def organize_name_data(self,year_data_dict):
        
        #need ids when inserting batch data into SQL tables
        #create them by using count_years and count_names
        name_sex_ids = {}
        year_popularity_ids = {}
        
        count_years = 1 
        count_names = 1
        for key,value in year_data_dict.items():
            year = key
            
            for entry in value:
                name,sex,popularity = self.get_name_info(entry)
                if (name,sex) not in name_sex_ids:
                    name_sex_ids[(name,sex)]=count_names
                    count_names+=1
                year_popularity_ids[(year,popularity,name_sex_ids[(name,sex)])] = count_years
                count_years += 1
        return name_sex_ids, year_popularity_ids
    
    def get_name_info(self,name_entry):
        name = name_entry[0]
        sex = name_entry[1]
        popularity = name_entry[2]
        return name, sex, popularity

    def prep_dict_4_SQL(self,names_dict,years_dict):
        names_prepped = []
        years_prepped = []
        for key, value in names_dict.items():
            names_prepped.append((value,key[0],key[1]))
        for key, value in years_dict.items():
            years_prepped.append((value,key[0],key[1],key[2]))
        return names_prepped, years_prepped
        
    def batch_insert_data(self,names_prepped,years_prepped):
        try:
            msg_names = '''INSERT INTO names VALUES(?,?,?) '''
            msg_years = '''INSERT INTO popularity VALUES(?,?,?,?) '''
            self.execute_commit_msg(msg_names,names_prepped,many=True)
            self.execute_commit_msg(msg_years,years_prepped,many=True)
            return True
        except Error as e:
            print("Database Error occured: {}".format(e))
        finally:
            self.conn.close()
        return None
```

All of the code can be found <a href="https://github.com/a-n-rose/recommendation-systems-python/blob/master/babyname_recommender/data_prep_babyname.py">here</a>. 


## Next Steps

I will play around with the new database, and see how I like I set it up. I will also probably start on making a basic recommender. Probably in parallel I will generate other features to work with and brainstorm all the kinds of features I want in such a recommendation system. 

Keep tuned!
