---
layout: post
title: "Babyname Recommender: Building a Recommender in Python Using KMeans Clustering"
date: 2018-11-16
---

I started building <a href="https://github.com/a-n-rose/recommendation-systems-python/tree/master/babyname_recommender">this recommender</a> while I was pregnant with my daughter, with no idea what to name her. I really wanted to know what other names were out there that I might also like. I am satisfied with how the project turned out (it did help my husband and me find a name) but there are some improvements that can be made to make it run more efficiently and also of course improve on recommendations. But, for the most part, I'm happy with it!


## Bulding the Recommender

To build my recommender, I decided to use SKlearn's KMeans Clustering. I figured that because I'm using the <a href="http://www.internationalphoneticalphabet.org/ipa-sounds/ipa-chart-with-sounds/">International Phonetic Alphabet</a> (IPA) features and classifications, which basically cluster consonantal and vowel sounds, the names could be clustered too. (Note: I also want to try K-Nearest-Neighbors.) 

But before I get there, I'll walk through the features I used to build clusters.

## Prep Features for Clustering

The IPA seemed to me the best choice to use as a base for features because it represents sounds much more accurately than English letters do.

First of all, English letters change in how they sound depending on the context they are found. For example, let's examine the letter *a* and how it changes in the phrase:
```
"Adam and Julia ate oranges." 
```
Does it maintain the same sound throughout? Clearly not. The international phonetic alphabet (IPA) is a useful tool to show these differences. (You can generate your own IPA transcriptions <a href="https://tophonetics.com/">here</a>.)

```
"Adam and Julia ate oranges."

ˈædəm ænd ˈʤuljə eɪt ˈɔrənʤəz.
```

Notice the different IPA characters used for *a*? The English letter *a* is represented here as all of the following IPA characters: *æ*, *ə*, and *eɪ*.

Additionally, the IPA uses markings denoting primary and secondary stress. Above you can see *Adam*, *Julia*, and *organges* have primary stress markers at the start. Below I'll show you a sentence with secondary as well as primary stress markers at various locations:

```
"Elizabeth likes covering balloons with newspaper cuttings."

ɪˈlɪzəbəθ laɪks ˈkʌvərɪŋ bəˈlunz wɪð ˈnuzˌpeɪpər ˈkʌtɪŋz.
```

Granted it's a bit of an odd sentence, it shows how the IPA markings can be used to mark differences in stress: the name *Elizabeth* has the most stress after the letter *l*, *baloons* does as well, and *newspaper* has two stress markers, a primary stress marker at *news* and a secondary stress marker at the start of *paper*.

I personally like names largely depending on the sounds they contain and the types of stress they have. 

Examples of names (for girls) I love or dislike:
```
LOVE:

Evelyn    --> ˈɛvələn

Madeleine --> ˌmædəˈlɛn

Danielle  --> ˌdæniˈɛl


DISLIKE:

Arabella  --> ˌærəˈbɛlə

Valeria   -->  vəˈliriə

Gabriela  -->  gɑbriˈɛlə
```

Some patterns I notice here are that I *like* names that end with a voiced consonant more than those that end with a vowel. I also prefer names that don't have plosive bilabials, i.e. *b*s but I love names that have alveolar consonants like *l*s, *d*s, and *n*s (notice that the tongue is in basically the same place when pronouncing each of those letters). Lastly, I seem to prefer names with either primary or secondary stress on the first syllable. Note: I can't say I agree with the IPA transcription of *Madeleine*. I think the first syllable has the primary stress, not the last. But, overall the IPA transcriptions are quite good. To see how the IPA categorizes the individual letters, this <a href="http://www.internationalphoneticalphabet.org/ipa-sounds/ipa-chart-with-sounds/">site</a> is a great reference.

Another reason why IPA characters offer more reliable features than English letters is in how they represent the length of sounds. The length of names based on English letters alone can be misleading. Compare the names *Ashleigh* and *Cornelia* and their character length using either English or IPA characters. 

```
Ashleigh

ˈæʃli 

--------

Cornelia 

kɔrˈniljə

```

You can see that spelling differences in English can deceive an algorithm into categorizing those two names as having the same length, even though *Cornelia* sounds much longer than *Ashleigh* in reality. The IPA characters, 5 vs 9 (including stress markers) represent the name lengths more more accurately.

In sum, using IPA characters, including the stress markers, to categorize names based on sound would be more useful as features than English characters alone. 

## Transform Features

Espeak is a framework that can transcribe several languages into IPA characters. Once I collected the names from the dataset, I created a pandas dataframe to add IPA features.

Note: to run this code, you need to have Espeak installed on your machine. Also, it took my computer **a half-hour** to transcribe all the names

```
import pandas as pd
from subprocess import check_output

def name2ipa(name_dict):
    '''
    espeak transcribes names into ipa_chars
    this takes a while
    '''
    start = time.time()
    name_df = pd.DataFrame.from_dict(name_dict,orient="index",columns=["name"])
    name_df["ipa"] = name_df["name"].apply(lambda x: check_output(["espeak","-q","--ipa","-v","en-us",x]).decode("utf-8"))
    print("{} hours".format((time.time()-start)/3600.))
    return name_df
```

Now the names and transcriptions are in a pandas dataframe, it's really easy to create feature colums.

First, in order to make columns for my features, I need to know the letters and lengths of all the names, in ipa characters of course.

```
def collect_used_ipa(name_df):
    '''
    Collect all IPA chars used in the names dataset
    Collect the various lengths of IPA --> useful in features
    '''
    ipa_chars = []
    ipa_lengths = []
    for index, row in name_df.iterrows():
        ipa_chars.append(list(row[1]))
        ipa_len = len(row[1])
        if ipa_len not in ipa_lengths:
            ipa_lengths.append(ipa_len)
    #flatten the list and put it in order
    ipa_chars = sorted(set([item for sublist in ipa_chars for item in sublist]))
    #remove irrelevant characters:
    ipa_chars = [char for char in ipa_chars if char not in ["(",")","-","\n"," "]]
    return ipa_chars, ipa_lengths
```

To generate useful features from these transcriptions, I collected all the IPA chars used and saved them in a separate file. In this file, I categorized all the IPA characters according to the IPA charts.


As examples, below is code for where I stored all the possible IPA chars, vowel types, and stress symbols. The consonant information is in the repo. (Reference <a href="http://www.internationalphoneticalphabet.org/ipa-sounds/ipa-chart-with-sounds/">here</a> for how I categorized the characters )
```
def ipa_characters():
    chars = ('a', 'b', 'd', 'e', 'f', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'r', 's', 't', 'u', 'v', 'w', 'x', 'z', 'æ', 'ð', 'ø', 'ŋ', 'ɐ', 'ɑ', 'ɔ', 'ə', 'ɚ', 'ɛ', 'ɜ', 'ɡ', 'ɣ', 'ɪ', 'ɬ', 'ɹ', 'ɾ', 'ʃ', 'ʊ', 'ʌ', 'ʒ', 'ʔ', 'ˈ', 'ˌ', 'ː', '̩', 'θ', 'ᵻ')
    return chars

def ipa_vowels_dict():
    ipa_vowels = {}
    ipa_vowels["open_vowels"] =('æ','ɐ','a','ɑ')
    ipa_vowels["mid_vowels"]  = ('e','o','ø','ɔ','ə','ɚ','ɛ','ɜ','ʌ')
    ipa_vowels["close_vowels"] = ('i','u','ɪ','ʊ','ᵻ')

    ipa_vowels["front_vowels"] = ('i','ɪ','e','ø','ɛ','æ','a')
    ipa_vowels["central_vowels"] = ('ᵻ','ə','ɜ','ɐ',)
    ipa_vowels["back_vowels"] = ('u','ʊ','o','ʌ','ɔ','ɑ',)

    ipa_vowels["rounded_vowels"] = ('u','o','ɔ')
    
    return ipa_vowels
    
def ipa_stress_dict():
    ipa_stress = {}
    ipa_stress["primary_stress"] = 'ˈ'
    ipa_stress["secondary_stress"] = 'ˌ'
    ipa_stress["long_vowel"] = 'ː'
    ipa_stress["english_button"] = '̩'
    return ipa_stress

```


Long story short, I used these features to generate thousands. 

### Pairs of Sounds

I found *pairs* of sounds very relevant. Usually I like sounds with some sounds but not others. For example, separately, I like *e* as in *see* as well as *a* as in *opera*. BUT, *i* + *a* at the ends of names? For some reason I detest them. I want my recommender to be able to recognize that. Therefore, I created a feature for every possible pairing (repeats excluded) of the IPA characters. 

### IPA Sound Categories

The IPA offers rich detail about where each sound is generated in the mouth and what type of sound it is. For each of these details - relevant to this name database - I created a feature. For example, *Danny* would have the features *alveolar* for both *d* and *n*, *front_vowel* and *open_vowel* for *a* and *front_vowel* and *close_vowel* for *y*. (I did not keep track of how many "front_vowels" were in the name, just that they were in it.) Additionally, *voiced* would be indicated as a feature and *voiceless* as not; this is relevant because a lot of names have *voiced* sounds but fewer have *only* voiced sounds. 

Compare *Danny* with *Phillip*. *Danny*, as I mentioned, has no *voiceless* consonants. *Phillip*, though, does. The *ph* does not require vocal cord activation, and neither does the *p* on the end. The *l*s do though. This can tell the algorithm if the user prefers only voiced, a mix, or rather only voiceless (e.g. *Kate*, *Stacey*, *Pete*).

### Stress and Length

Some people like strong short names, long, delicate ones, and all mixes inbetween. I used the primary and secondary stress markes to indicate whether a name had more stress on the first half of the name, like *Harry* or *Elizabeth* vs stress on the latter half, as in *Josephina* or *Gerome*. 

That basically sums it up. To see how I generated these features, refer to the <a href="https://github.com/a-n-rose/recommendation-systems-python/blob/master/babyname_recommender/ipa_features.py">code</a>.

## Building Clusters

With all of these features, the irrelevant columns removed (e.g. if no names had certain IPA pairs), I had over 1,000 features. I wanted to remove the features that were not relevant to the user before building clusters. Therefore, in my application, before the user could get recommendations, they *had* to rate names. Note: they can create search instances and see only boy or girl names, or all names.

Once the user has rated names they liked *as well as* names they didn't, the application selected features via recursive feature elimination (RFE). Only the features that helped the solver predict the names a user would like would be selected. Note: names the user liked were weighted in this process. 

The clusters would then be fit on the name features, but only the features that are relevant to that user, not the 1,000+ features I had to start out with. 

After each rating session the user performs, the clusters associated to that specific search instance get updated.

## Trying it Out:

I already mentioned some sounds I like in names. Let's see what this recommender does for me as I rate names. 

Girl names:

```
30 names rated

Relevant features: eɪ, ɪl, open_vowels, voiceless, fricative

This recommender found names you liked 28.000000000000004%  of the time

Names I ended up saving this round:

Reyon
Ravion
Almae
Deyanara
Dorann

--------------


80 names rated

Relevant features: eɪ, ɪd, ɪj, ɪˈ, postalveolar

This recommender found names you liked 8.0%  of the time

Names I ended up saving this round:

Zaydah

```

The "Relevant features" the recommender identified for me will likely change quite a bit until I have rated enough names. I also may change the number of selected features; when I used more, however, I didn't see enough "focus" in my recommendations. I like what I was recommended more when I set a stricter limit. 

I will fine-tune this recommender from time to time, perhaps explore K-Nearest-Neighbors or improving my code to be more efficient. Hopefully this can be of interest to someone out there :P 
