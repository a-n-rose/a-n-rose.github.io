---
layout: post
title: "Save to SQLite3 Database with Python: Why Dictionaries Kick-Ass"
date: 2018-11-06
---

While putting together my new-and-improved <a href="/2018/11/05/updated-babyname-recommender.html">babyname recommender</a>, I saw the opportunity to further explore the advantages and disadvantages of techniques for saving data to a database. To see the code I show below in its full context, check out my <a href="https://github.com/a-n-rose/recommendation-systems-python/tree/master/babyname_recommender">repo</a>.

In my scenario, I am working with an SQL database to insert and save the names, sex, and year popularity of all names from USA social security applications, ranging from the years 1880 until 2017. (The name data can be downloaded from <a href="https://catalog.data.gov/dataset/baby-names-from-social-security-card-applications-national-level-data">data.org</a>.)

The issue is that I need to have a table with only the used names and their assigned sex - no repeats. And I need another table with the years, the popularity of a name, and that name's ID number. 

This means that I nead a unique ID number assigned to each name in the table. I can either let SQL do this for me, which I do when I insert the data at each iteration. OR I can create the IDs myself, which python dictionaries allow me to do beautifully, shortening the processing time **drastically**

## Iterative Insert and SQLite3

The first technique I implemented did a pretty good job not taking over my CPU. When I ran it, my CPU mostly remained at around 20% - 25%, with the rare occasion going above 30%. This meant that I could use my computer as usual, while my program collected, organized, and safely saved my data for me. 

The time to go through all the data, however, took forever. This is because data was getting inserted and commited to the database at every iteration. That slows the program way down.


```
def save_name_data_2_SQL(self):
    #collect filenames
    text_files = self.collect_filenames()
    num_years = len(text_files)
    for text_path in text_files:
        year_start = time.time()
        with open(text_path) as f:
            data = f.read()
        year = self.get_year(text_path)
        print("Processing names in the year {}".format(year))
        name_data =  self.separate_name_data(data)
        
        
        
        ###### BELOW IS THE CRAZY ITERATION #######
        
        
        for entry in name_data:
            self.insert_name_data(entry,year)
            section = 'names of {}'.format(year)
            
            
    return None

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

To give you an idea, this was the progress when I finally pulled the plug:
```
50.0% through all years
Program running for 12.88 hrs
Processing names in the year 1949
```

I pulled the plug well after my dictionary solution; I just wanted to see how long it would take! But after almost 13 hours and only 50% progress... I couldn't let it keep going. I put it out of its misery.

## Dictionaries --> Batch Insert

Instead of inserting data iteratively into the database, I iteritavely added each year (with its data in a list) to a python dictionary. The benefits of this were that the data would be immutable and that dictionaries could add the data very quickly. 

Below is how I adapted my code from the function I above: 'save_name_data_2_SQL':
```
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
        name_data =  self.separate_name_data(data)
        
        ###### INSTEAD OF ITERATING, SAVE TO DICT #######
        
        year_data_dict[year] = name_data
    return year_data_dict
```
From there I wasn't sure how to create unique IDs for the names in order to link the tables 'names' and 'popuarity'. In my for-loop, SQL filled in the ID values for me, linking the table without me having to do much. 

Shortly though, I found an easy solution: count += 1

```
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
            
            #avoid name duplicates
            if (name,sex) not in name_sex_ids:
                name_sex_ids[(name,sex)]=count_names
                count_names+=1
                
            year_popularity_ids[(year,popularity,name_sex_ids[(name,sex)])] = count_years
            count_years += 1
    return name_sex_ids, year_popularity_ids
```
From the dictionaries I returned, I could easily make a list of tuples to batch insert into SQL. 

```
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

This was so much faster than the iterative inserting of data. It took a mere 40 seconds to collect, organize and save the data to the SQL database.
