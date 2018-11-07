---
layout: post
title: "Putting Python to the Unittest: Classes and Sqlite3"
date: 2018-11-07
---

Much of my previous work (and probably some of my future experimental work) did not include much testing. That is a habit I would like to change, and I'm not waiting for New Years to do that. 

My most recent app (as of Nov. 2nd 2018) is my vocab/idea collector. Since this could theoretically be built into a real functioning app, unittest development simply is a must. (For code: <a href="https://github.com/a-n-rose/Vocab-Collector-and-Tester">REPO</a>; for a <b>working</b> blogpost: <a href="/2018/11/05/personal-vocab-app.html">BLOG</a>)

To see the unittests I made for this app, the code can be viewed <a href="https://github.com/a-n-rose/Vocab-Collector-and-Tester/blob/master/test_vocab_collector.py">here<a>.

## Unittest development

In order to build the right unittests, I needed to identify and sort the kind of functions I had.

1) stand-alone input-output functions

2) SQLite3 related functions --> adding or retrieving data from a database

3) functions within a class using class attributes

4) functions handling user input

## Write the code

I listed the functions types above in the order of difficulty / anxiety I had for them. 

The first ones were easy.

### Unittesting Stand-Alone Functions

I only had one truely stand-alone function. This function simply took a user's input and removed any spaces and special characters. Here is the test for that:
```
import unittest
#import input_manager module, my script holding the function(s) I want to test
import input_manager


class VocabInputCheck(unittest.TestCase):

    def test_rem_space_specialchar_nospace_nospecialchar(self):
        user_input = 'Stacey'
        self.assertEqual(input_manager.rem_space_specialchar(user_input),'Stacey')
        
    def test_rem_space_specialchar_withspace_nospecialchar(self):
        user_input = 'S t a c e y'
        self.assertEqual(input_manager.rem_space_specialchar(user_input),'Stacey')
        
    def test_rem_space_specialchar_nospace_withspecialchar(self):
        user_input = '$()(S?@ta______c&!e+=y'
        self.assertEqual(input_manager.rem_space_specialchar(user_input),'Stacey')
        
    def test_rem_space_specialchar_noalphanumeric(self):
        user_input = '*#(!)  (# #&___$$(@ '
        self.assertEqual(input_manager.rem_space_specialchar(user_input),None)
        
    def test_rem_space_specialchar_empty(self):
        user_input = ''
        self.assertEqual(input_manager.rem_space_specialchar(user_input),None)
        
if __name__ == '__main__':
    unittest.main()
```

In these tests, I put in all sorts of things a user could enter. I checked that my function handled such input as expected, which I 'assert' to be true in the self.assertEqual() function. 

All the other functions seemed to be largely dependent on data in the vocab list database. 

### Unittesting SQLite3 Functions 

This was actually easier than I expected. 

All I had to do was create a test_database to do my tests on, and then delete that database when I was done. I can do that with setUp() and tearDown(). 

Note: I create a database through creating an instance of my class 'Collect_Vocab'. You can create a connection on your own with the following code:
```
self.conn = sqlite3.connect('database.db')
self.c = self.conn.cursor()
```
Or something similar. 

Here's my setUp() function:

```
import unittest
import sqlite3
import os


#import class from module with my functions:
from vocab_collector import Collect_Vocab


class TestUserVocabDatabase(unittest.TestCase):
    '''
    Test the user vocab collection database
    '''
    
    def setUp(self):
        '''
        Setup a temporary database
        '''
        #create database
        self.db = Collect_Vocab('test_vocab.db')
    
    
    def tearDown(self):
        if self.db.conn:
            self.db.conn.close()
        os.remove("test_vocab.db")
```

In the setUp() function, I initialize my class (which I imported from my module). This in turn initializes a database, which I named 'test_database.db'. Now that's up, I need to create the tables necessary for my app.

(This code gets added to the setUp() function above)

```
        #create necessary tables
        msg = '''CREATE TABLE IF NOT EXISTS users(user_id integer primary key, username text, password text) '''
        self.db.c.execute(msg)
        self.db.conn.commit()
        
        msg = '''CREATE TABLE IF NOT EXISTS vocab_lists(list_id integer primary key, list_name text, list_number int, tags text, list_user_id integer, FOREIGN KEY(list_user_id) REFERENCES users(user_id) ) '''
        self.db.c.execute(msg)
        self.db.conn.commit()
        
        msg = '''CREATE TABLE IF NOT EXISTS words(word_id integer primary key, word text, meaning text, example_sentence text, tags text, word_list_id integer, FOREIGN KEY(word_list_id) REFERENCES vocab_lists(list_id)) '''
        self.db.c.execute(msg)
        self.db.conn.commit()
```

The tables are ready for me to fill in with some test data. This allows me to create possible situations someone could use my app for and see if my app handles it appropriately. 

First the users:
```
        #insert some data
        user1 = ('1','Stacey','sailboat')
        user2 = ('2','Freddy','fidgetspinner')

```

Then some vocabulary lists they have:
```
        #list name can also be included in 'tag search'
        user1_vocablist1 = ('1','German','1','every-day; beginner','1')
        user1_vocablist2 = ('4','Body Parts','2','German;intermediate','1')
        
        #can quiz user on multiple lists if they have the same name
        user2_vocablist1 = ('2','Colors','1','French;colors; beginner','2')
        user2_vocablist2 = ('3','Colors','2','Arabic;colors; beginner','2')

```

And finally some words and meanings etc. filled into the lists. 

I decided to play with a few different languages, even Arabic (eek!). Also, I left the 4th list empty to make sure my app handles that appropriately.
```
        vocablist1_words1 = ('1','Haus','house','Das Haus hat drei Schlafzimmer und zwei Badezimmer, perfekt für meine Familie; Er wollte sein Haus verkaufen weil es, ohne seine Kinder, zu groß war.','lifestyle; housing; accommodation','1')
        vocablist1_words2 = ('4','Frühstück','breakfast','Mein Magen morgens mag kein Frühstück; Zum Frühstück findet sie Pfannkuchen mit Ahornsirup am besten.','lifestyle; food; daily tasks','1')
        vocablist1_words3 = ('8','Büro','office','Meine Mama ist gerade im Büro weil sie einen Termin mit ihren Arbeitskollegen hat; Im Büro liegen alle meine Arbeitsunterlagen.','work; rooms;','1')
        
        vocablist2_words1 = ('2','bleu','blue',"Aujourd'hui, il n'y a pas de nuages et le ciel est bleu.;Il est ennuyeux que les vêtements d'enfants soient en rose ou en bleu. Il y a tellement d'autres couleurs là-bas!",'French; colors; easy','2')
        vocablist2_words2 = ('3','jaune','yellow',"Voici le bus scolaire jaune vif.;Le soleil émet en réalité plus de lumière verte que de lumière jaune.",'French; colors; intermediate','2')
        #to test the vocab app to find similar words in a sentence:
        #rouge --> rouges
        vocablist2_words3 = ('7','rouge','red',"Le ketchup a laissé plusieurs taches rouges sur le tapis.;Ses yeux sont rouges parce qu'il n'a pas dormi la nuit dernière.",'French; colors; plural; intermediate','2')
        
        
        #show limitations of app with non latin alphabets
        vocablist3_words1 = ('5','أزرق','blue',"السماء الزرقاء الصافية جميلة.",'Arabic; colors; easy','3')
        #The clear blue sky is beautiful. 
        vocablist3_words2 = ('6','الأصفر','yellow',"الشمس صفراء; الموز الأصفر",'Arabic; colors; intermediate','3')
        #the sun is yellow
        #bananas are yellow
        vocablist3_words3 = ('9','أحمر','red',"الدم ليس أزرق ولكن أحمر.;الورود هي حمراءالبنفسج هي زرقاء",'Arabic; colors; plural; intermediate','3')
        #the blood is red not blue
        #roses are red violets are blue
        
        #vocablist4 will remain empty
```
And then save all that into the corresponding tables in the test database:
```
        
        msg = '''INSERT INTO users VALUES (?,?,?) '''
        self.db.c.executemany(msg,[user1,user2])
        self.db.conn.commit()
        
        msg = '''INSERT INTO vocab_lists VALUES (?,?,?,?,?)'''
        self.db.c.executemany(msg,[user1_vocablist1,user1_vocablist2,user2_vocablist1,user2_vocablist2])
        self.db.conn.commit()

        msg = '''INSERT INTO words VALUES (?,?,?,?,?,?) '''
        self.db.c.executemany(msg,[vocablist1_words1,vocablist1_words2,vocablist1_words3,vocablist2_words1,vocablist2_words2,vocablist2_words3,vocablist3_words1,vocablist3_words2,vocablist3_words3,])
        self.db.conn.commit()
```

From there I could start testing my functions that need such data! I will leave one example test to give an idea of how they look:

To test if my app correctly identifies if an entered password matches the one in the database, I made a test for that:

```
    def test_check_password_correct(self):
        username = 'Stacey'
        password = 'sailboat'
        self.assertEqual(self.db.check_password(username,password),True)
    
    def test_check_password_incorrect(self):
        username = 'Stacey'
        password = 'sailboats'
        self.assertEqual(self.db.check_password(username,password),False)
```

Here is another test making sure my app correctly creates a new vocabulary list. I need a 'user_id' in order to look up the user's list, and because I have an active test class instance, I can define the user_id to the one I want. Here I define it as '2', which corresponds to the fake user 'Freddy'.

To ensure the newly list wasn't there to begin with, I make sure that the lists collected from the user *before* the list is added, does not contain the list that will be added.

```
    def test_create_new_list(self):
        list_name = 'Colors'
        tags = 'Spanish; colors; easy'
        self.db.user_id = 2
        self.assertEqual(self.db.coll_user_vocab_lists(),[(2,'Colors',1,'French;colors; beginner',2),(3,'Colors',2,'Arabic;colors; beginner',2)])
        self.assertEqual(self.db.create_new_list(list_name,tags),None)
        self.assertEqual(self.db.coll_user_vocab_lists(),[(2,'Colors',1,'French;colors; beginner',2),(3,'Colors',2,'Arabic;colors; beginner',2),(5,'Colors',3,'Spanish; colors; easy',2)])
```
For more tests, please refer to the <a href="https://github.com/a-n-rose/Vocab-Collector-and-Tester/blob/master/test_vocab_collector.py">code</a>.

### Unittesting Functions within Classes

After setting up the test database and testing the functions with that, I had inadvertently also figured out how to apply unittests to my functions within classes. 

I simply created an instance of my class within the test, and could test the functions that way, creating whatever variables I needed for that class as I need. For example, when I created the 'user_id' variable. Because I was able to test just single functions from within the class, I didn't have to worry about messing anything else up with the fake variable I assigned to self.user_id. 

As I mentioned earlier, I created my class instance in the setUp() function.

```
    def setUp(self):
        '''
        Setup a temporary database
        '''
        #create database
        self.db = Collect_Vocab('test_vocab.db')
```
That is how you can create an instance for any class you want to test. 

### Unittesting user input()

To be honest, I haven't gotten that far. I've played around with mocking input, but don't quite get how that works yet. So... I hope to update this post with what I find pretty soon. But at least I've got most of the other unittests started!

## Next Steps

I will need to fine-tune the tests, making sure they raise the expected errors or can handle various kinds of tasks. This will be a work in progress. 
