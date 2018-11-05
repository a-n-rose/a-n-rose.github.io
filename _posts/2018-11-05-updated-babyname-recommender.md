---
layout: post
title: "Babyname Recommender: Setting up the Database with SQLite3"
date: 2018-11-05
---

Don't get any ideas; I'm not pregnant nor am I planning to be. This is just a really fun project that I built in the past (while I was pregnant). 

Back then I had the goal to understand frontend better. I learned a lot but I'm happy to further explore this recommendation system in just the backend world.

The repo and code can be viewed <a href="https://github.com/a-n-rose/recommendation-systems-python/tree/master/babyname_recommender">here</a>.

## Collect the names

The most updated name list can be found at <a href="https://catalog.data.gov/dataset/baby-names-from-social-security-card-applications-national-level-data">data.org</a>. It is a collection of names from USA social security applications, dating from 1879/1880 until, as of writing this, 2017. 

I unzipped the file into a subdirectory of my current working directory 'names'. (Note: the size of the unzipped contents is appx. 24 MiB.)

## Build a Database

This sounds easy but it does require some extra thought. When creating a database, it is easier in the long-run to build a database with sufficient feature columns as well as tables (and there can be many) than to build a database with insufficient columns and tables, only to have to add/change them later. I at least don't have fun doing that, so I try to avoid that. 

For my name database, I need to think about the possible functionality. For example, what I did last time was I retained only the most recent use of a name. That means that I didn't keep how popular the name 'Anna', for example, was in the year 1903. I can imagine someone (myself included) being interested in exploring names popular in a specific year, and if I store data that way, I can't do that. 

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

### Plan the database

Looking at this and knowing I want to keep as much information in the database as possible I will build a database with the following structure:

1) table with all names and assigned sex --> name_id; name; sex; 
2) table for all years and the popularity of each name in that year --> year_id; year; popularity; name_id (links to the name/parent table)

I can add additional tables later, for example tables with phonological information or name meanings (e.g. dream, protector) and backgrounds (e.g. Arabic, Irish).

While in the long-run I'd prefer to work with SQLAlchemy, I will build this first version with SQLite3 as it comes preinstalled in Python3+. My goal is to make this as starter friendly as possible. 

## Write the Code: General Tips

1) 

One key thing to remember when setting up code to save data to a database is to ensure you can **appropriately close the database**, despite any errors that may come up. For that, common practice in Python is to use the 'try statement'. The 'try statement' includes also 'except' and, optionally, 'finally' statements (see <a href="https://www.w3schools.com/python/python_try_except.asp">here</a>).

Basically it tries some code and if it fails, it can get taken care of/ handled in the 'except' statement (let's say you don't want an error to stop your entire program). And in the 'finally' statement, anything else you want to be completed gets completed. Like, for example, appropriately closing your database so no data gets lost or messed up.

2)

Another helpful thing I've found is to **show progress**. There are some fancy options for that but I often times just code something like this:
```
total_data = len(data)
count=0
for sample in tot_data:
    count+=1
    #get a percentage rounded to 2 decimal places
    percent = round(count/float(total_data)*100,2)
    print("{}% through all data".format())
```
This spits out something like this at every iteration:
```
2.01% through all data
```
Yes, that means that my shell/screen is just full of lines like that above but... it helps me stay sane. It's on my ToDo list to improve. Don't judge. 

3) 

**Simplify functionality**. I'll be the first to admit it: it is fun to squeeze in as much functionality into one function. But it is a deep pimple to squeeze out if anything goes wrong. If you see your code exceeding 5 indendtations, seperate it into separate functions. That way your code is more understandable for others and yourself and you can also more easily apply unittesting if you so please.

## My code:

I separated my functionality into three modules:

1) main module that calls the others into action

2) function module that holds all the necessary functions

3) error module that holds any errors that I raise.

I can put out there the simpler modules: 
The main module (as of this writing called 'database_setup.py')

Notice the 'try statment' and in general how nice and simple this is.
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
        bn.save_name_data_2_SQL()
    except Exception as e:
        traceback.print_tb(e)
    finally:
        if bn.conn:
            bn.conn.close()
```

This code imports the more complicated, meaty module: 'data_prep_babyname', which is saved as data_prep_babyname.py.

That code I will break down first. 

It defines a class called 'Setup_Name_DB'. This class is initiated with a database name. It is programmed to search for text files and extract the data contained in them and save to an SQL database.

Before one can save data to an SQL database, you have to 

1) connect and create a database and

2) form the databse skeleton.

Below is code where I do that. First I import the necessary libraries.
I also import an error I defined in errors.py called 'MissingNameID'

Whenever Setup_Name_DB gets initiated, it creates/initializes a database. 

To form the skeleton, at least of the tables I want to insert data, I create the tables I need. When creating them, I need to specify the column names and the type of data they will hold. The tables I created here are the 'names' table and the 'popularity' table.

I also wrote a function 'execute_commit_msg'. I just got tired of entering the contents of this function in other functions... so I wrote a function for it.

```
import sqlite3
import numpy as np
import glob
import os

from errors import MissingNameID


class Setup_Name_DB:
    def __init__(self,database):
        self.database = database
        self.conn = sqlite3.connect(database)
        self.c = self.conn.cursor()
        
    def execute_commit_msg(self,msg,t=None):
        if t is None:
            self.c.execute(msg)
        else:
            self.c.execute(msg,t)
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

The next function 'save_name_data_2_SQL' is where the work gets done. It looks simple but that is because I broke this function down into several. In this code I call 4 other key functions, not counting a function showing progress.

```
    def save_name_data_2_SQL(self):
        #collect filenames
        text_files = self.collect_filenames()
        num_years = len(text_files)
        count_years = 0
        # I did a for loop here to reduce memory costs.
        for text_path in text_files:
            with open(text_path) as f:
                data = f.read()
            year = self.get_year(text_path)
            name_data =  self.separate_name_data(data)
            num_names = len(name_data)
            count_names = 0
            for entry in name_data:
                self.insert_name_data(entry,year)
                count_names+=1
                section = 'names of {}'.format(year)
                self.progress(count_names,num_names,section)
            count_years += 1
            self.progress(count_years,num_years,'all years')
        return None
```
All of the code can be found <a href="https://github.com/a-n-rose/recommendation-systems-python/blob/master/babyname_recommender/data_prep_babyname.py">here</a> but these functions above provide the necessary context. These are also what get called in the main function. 

Lest I forget, the errors. They are also really important. The reason why I coded an error is because inserting data into SQL databases is tricky. I don't want to spend 2 hours inserting data and because of one little error, which I could have handled, cause me extra hours work. 

In the function 'insert_name_data', I'm not always sure if there is data to insert or if that data already exists in the table. If I can skip work I don't have to do, I'll always skip it.

Find the function 'self.get_name_id'. I couldn't be sure if it existed in the names table, but it was imperative for inserting data about name popularity given a year. Therefore, if no name id could be found, I decided to raise a 'MissingNameID' error just in case. 

If I raise the error myself, I can make sure it doesn't stop my program. In my 'except' statment, I include a print statement (which should actually be a logging statement), stating that the name id wasn't collected and that that year's data for that name could not be saved.

```
    def insert_name_data(self,data_entry,year):
        try:
            name,sex,popularity = self.get_name_info(data_entry)

            #insert name and sex data into names database
            # IF name doesn't exist
            name_exist = self.check_if_name_exists(name,sex)
            if name_exist == False:
                t = (name,sex,)
                msg = '''INSERT INTO names VALUES(NULL, ?, ?) '''
                self.execute_commit_msg(msg,t)
            else:
                print("name {} for sex {} exists already".format(name,sex))
            #insert name popularity into years database:
            msg = '''INSERT INTO popularity VALUES(NULL, ?, ?, ?) '''
            name_id = self.get_name_id(name,sex)
            if name_id is None:
                raise MissingNameID("The name ID for the name {} for sex {} cannot be collected".format(name,sex))
            t = (str(year),str(popularity),str(name_id),)
            self.execute_commit_msg(msg,t)
        except MissingNameID as e:
            print(e)
            pass
        finally:
            self.conn.commit()
        return None
```

For reference, the 'errors' module looks like this:
```

class Error(Exception):
    """Base class for other exceptions"""
    pass

class MissingNameID(Error):
    pass
```
First the Error class had to be created, then I could create my own, inheriting from that Error class. 

## Next Steps

I have only inserted part of the data into the SQL database. Therefore currently I am still working on that. 

In addition I am brainstorming all the kinds of features I can create in order to improve my recommendation system. 

Keep tuned!
