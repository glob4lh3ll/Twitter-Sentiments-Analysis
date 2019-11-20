## The general steps I take to complete this project are:

- Get a twitter API and download Tweepy to access the twitter api through python
- Download twitter tweet data depending on a key word search “happy” or “sad”
- Format my tweets so that no capitalization, punctuation, or non ascii characters are present, as well as splitting the tweet into an array holding each word in a separate holder
- Create a bag of common words that appear in my tweets
- Create a frequency table of words that have positive and negative hits
- Test my frequency table by using test sentences

## Implementation

## Step 1 (Getting your twitter API key and installing Tweepy):
To use `twitter’s API`, we need to first create a twitter account. Afterwards, we go to `apps.twitter.com` and `create an app`.
After that go to `“Keys and Access tokens”` and get your API key and secret (copy and save them for later).
Then go and create your access token (you will find this by scrolling down), and then save your access token and access token secret.
These codes will allow us to access twitter’s API through python. Its pretty much the key needed to access twitter’s database.
Afterwards install `“Tweepy”` by using the script: `“pip install Tweepy”` in your terminal. Tweepy lets us interact with twitter more easily.

## Step 2 (Getting your data):
To get our data we will be using Twitter’s API and access it using the Tweepy library. Basically, we will authenticate our Twitter API using our access token, access secret, consumer key and consumer secret. Afterwards, we are going to have a variable where we store the phrase/word we want to query.
In essence, we find tweets that have our search query in them.
However, because accessing too many tweets in a short amount of time will throttle our program (twitter can’t allow us to use too much of their power), we have to set a timer on how fast we want to search our query. We also set a limit to the number of tweets we are looking for. I chose 15000.
So the first step was to import our libraries:

```bash
from tweepy.streaming import StreamListener
from tweepy import OAuthHandler
from tweepy import Stream
import tweepy
import json
import time
```

We then input our access token information and authenticate:

```bash
access_token = "-----------------"
access_secret = "----------------"
consumer_key = "----------------"
consumer_secret = "----------------"

auth = OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_secret)
 
api = tweepy.API(auth, wait_on_rate_limit=True, wait_on_rate_limit_notify=True)
```

If you notice on last line of above code. In the tweepy.API function I specify something called “wait_on_rate_limit_notify” and “wait_on_rate_limit” and set them to be True. Basically this tells my script to not close out of the program if I am being throttled by twitter. Instead, the script will wait until the throttling stops then resume the rest of my script. In this case, given sufficient time (around 3 hours), my script will be able to get around 15000 hits. The next thing we do is to enter our search query as well as how often we want our program to search and how many total searches:

```bash
#We begin searching our query
#Put your search term
searchquery = "angry"

users =tweepy.Cursor(api.search,q=searchquery).items()
count = 0
start = 0
errorCount=0

#We will be storing our data in file called: happy.json
#file = open('test.json', 'wb') 

#here we tell the program how fast to search 
waitquery = 100      #this is the number of searches it will do before resting
waittime = 2.0          # this is the length of time we tell our program to rest
total_number = 15000     #this is the total number of queries we want
justincase = 1         #this is the number of minutes to wait just in case twitter throttles us
```

Then the following is simply us iterating through and searching our query term many times over:

```bash
text = [0] * total_number
secondcount = 0
idvalues = [1] * total_number
 #1 is happy; 2 is sad; 3 is angry; 4 is fearful
#Below is where the magic happens and the queries are being made according to our desires above
while secondcount < total_number:
    try:
        user = next(users)
        count += 1
        
        #We say that after every 100 searches wait 5 seconds
        if (count%waitquery == 0):
            time.sleep(waittime)
            #break

    except tweepy.TweepError:
        #catches TweepError when rate limiting occurs, sleeps, then restarts.
        #nominally 15 minnutes, make a bit longer to avoid attention.
        print "sleeping...."
        time.sleep(60*justincase)
        user = next(users)
        
        
    except StopIteration:
        break
    try:
        #print "Writing to JSON tweet number:"+str(count)
        text_value = user._json['text']
        language = user._json['lang']
        #print(text_value)
        print(language)
        
        if "RT" not in text_value:
            if language == "en":
                text[secondcount] = text_value
                secondcount = secondcount + 1
                print("current saved is:")
                print(secondcount)

    except UnicodeEncodeError:
        errorCount += 1
        print "UnicodeEncodeError,errorCount ="+str(errorCount)
```

Pretty much I create an array called “text” in which I store the text values of the tweet, and create an index called secondcount which counts how many “tweets” I store. I did this because in file part1.py from lines 74 to 85, I made it so that I only stored texts that were in English and were not a retweet. The code works by having an index of users in a json format. So I can access the data for a given “user” by calling the user._json variable. So by stating text_value = user._json[‘text’] I can get the text file from the current user I am looking at. After I am done with whatever data I want from the user, in part1.py line 54 simply tells us that we want to go to the next user. Here, I believe “user” simply refers to a given tweet a user stated that includes our key word. I then transferred my arrays that had the user tweets in them, and transformed them into a dataframe and saved them as a csv file doing the following:

```bash
print("Creating dataframe:")

d = {"text": text, "id": idvalues}
df = pd.DataFrame(data = d)

df.to_csv('upset.csv', header=True, index=False, encoding='utf-8')

print "completed"
```

I did the above program and used the terms:
1. #happy
2. #fun
3. #sad
4. : (
As my labels for the feelings of: happy and sad feelings.

## Step 3 (Cleaning the data and getting the words that appear):
If you have followed what I have done till now and checked your csv files you will notice that some of the tweets have weird symbols. Our first goal is to get rid of them. Afterwards, we want to produce a csv file with the words that appear in our sentences.
To clean the data, the first thing we do is to import any libraries we need and import the csv we are interested in as well getting rid of any “nan” values (in part2.py line 10 does this).

```bash
import pandas
import string

#import my csv file
df = pd.read_csv('unsmile.csv')

#Remove any rows with a "nan" in them
df = df.dropna(axis=0, how = 'any')
```

We then create a function that, given a text, removes any character or string of characters that are not readable in ASCII values. We then make all the texts lower case. I use df[‘text’] because that is the name of the column I stored the text values in the csv file.
```bash
#Make it so that any non readable text gets converted into nothing
def removetext(text):
    return ''.join([i if ord(i) < 128 else '' for i in text])

#Here I am doing the actual removing
df['text'] = df['text'].apply(removetext)

#Make all my texts lower case
df['text'] = df['text'].apply(lambda x: x.lower())
```

The df[‘text’].apply(lambda x: x.lower()) basically tells us to apply the function lambda x: x.lower onto each row in the dataframe. (lambda x: x.lower()) simply states that given an input “x” make all the characters in it lower case.
Using the same format, we can remove any unwanted punctuation.

```bash
#Get rid of all weird punctuation and extra lines
df['text'] = df['text'].apply(lambda x: x.replace('.',' '))
df['text'] = df['text'].apply(lambda x: x.replace('\n',' '))
df['text'] = df['text'].apply(lambda x: x.replace('?',' '))
df['text'] = df['text'].apply(lambda x: x.replace('!',' '))
df['text'] = df['text'].apply(lambda x: x.replace('"',' '))
df['text'] = df['text'].apply(lambda x: x.replace(';',' '))
df['text'] = df['text'].apply(lambda x: x.replace('#',' '))
df['text'] = df['text'].apply(lambda x: x.replace('&amp',' '))
```

With that our dataframe is clean.
Our next goal is to get the unique words from it that appear.

```bash
#Here I get each unique keyword from my dataframe
array = df['text'].str.split(' ', expand=True).stack().value_counts()
#print(array) to see what this looks like

#I make a dataframe of the words and the frequency with which the words appear 
d = {'word': array.index, 'frequency':array}
df2 = pd.DataFrame(data = d)

#I get rid of any words that are mentioned less than 10 times
df2['frequency'] = df2['frequency'][df2['frequency'] > 10] 
```

Line 34 does two things. It first splits every string in each row of the dataframe into individual words. It does this by splitting after every space. Afterwards, “.stack().value_counts()” finds a new/unique word and counts how many times it appears. It stores these values into the variable called “array.”
Line 38 makes a dictionary of the frequency with which each unique word appears, and what the unique word is. The unique word string is stored in the array.index variable.
Line 39 makes the dictionary into a dataframe.
Line 42 then checks each value in the “frequency” column and replaces any values than 10 with a Nan.
The ending of the script then looks like:

```bash
#Remove any rows with a "nan" in them
df2 = df2.dropna(axis=0, how = 'any')

#Drop any obvious signs of these words being :(
df2 = df2.drop([':(','https://t',':((', ':(((', ':((((', ':(((((', ':', '(', ''])

#Convert my dataframe into a csv file
df2.to_csv('unsmile_words.csv', header=True, index=False, encoding='utf-8')
```

My goal in line 45 was to drop any of the rows with a Nan value.
I then dropped any rows with the keywords shown in line 48 because I didn’t want my program to have words it was biased to that were too obvious because they included the word I searched for.
Line 51 converts the dataframe into a csv file. I repeated this for all of my previous csv files.

## Step 4 (Get the bag of words):
Now that I have the list of words that appear in each individual file (happy, fun, sad, and : ( ), I want to combine them all into one dataframe and save this into a csv file called wordbag.csv.

```bash
import pandas as pd

#import my csv files
happy = pd.read_csv('happy_words.csv')
sad = pd.read_csv('sad_words.csv')
unsmile = pd.read_csv('unsmile_words.csv')
fun = pd.read_csv('fun_words.csv')


wordbag = pd.concat([happy,sad,unsmile,fun]).drop_duplicates(subset = 'word').reset_index(drop=True)

print(wordbag)

wordbag.to_csv('wordbag.csv')
```

Like always, I start by importing the pandas library.
I then read the csv files with the words into the script.
Line 11 then combines all of the dataframes into one. That’s what the concat does. Afterwards, the drop_duplicates gets rid of any word that appears multiple times. I also reset the index.
Line 15 then saves this into a wordbag.csv file.
The reason why I wanted a bag of words was so that I could do a frequency count. Or in other words, I wanted to see how many of the “happy” tweets mentioned a given word in the bag of words, for all of the words. I would do the same for the sad, fun, and : ( tweets too.



In essence I wanted to see that out of the happy tweets, how many of them mentioned the word “me”, and in this case it would be 20. I would do this across all of my classifiers and all of the words that were interesting to me. The words that are interesting to me, as you know, is the bag of words we generated.

## Step 5 (Split the tweets into individual words):
To make it easier on us. I went back to our tweet files (happy.csv, sad.csv, unsmile.csv, and fun.csv) and made it so that each column would include the list of words that appeared in the tweet as an array: 


So each row will be a tweet, and in the text column is an array of the words that appeared in the tweet. I then save this as a dataframe.

```bash
import pandas as pd




df = pd.read_csv('unsmile.csv')

#Remove any rows with a "nan" in them
df = df.dropna(axis=0, how = 'any')

#Make it so that any non readable text gets converted into nothing
def removetext(text):
    return ''.join([i if ord(i) < 128 else '' for i in text])

#Here I am doing the actual removing
df['text'] = df['text'].apply(removetext)

#Make all my texts lower case
df['text'] = df['text'].apply(lambda x: x.lower())

#Get rid of all weird punctuation and extra lines
df['text'] = df['text'].apply(lambda x: x.replace('.',' '))
df['text'] = df['text'].apply(lambda x: x.replace('\n',' '))
df['text'] = df['text'].apply(lambda x: x.replace('?',' '))
df['text'] = df['text'].apply(lambda x: x.replace('!',' '))
df['text'] = df['text'].apply(lambda x: x.replace('"',' '))
df['text'] = df['text'].apply(lambda x: x.replace(';',' '))
df['text'] = df['text'].apply(lambda x: x.replace('#',' '))
df['text'] = df['text'].apply(lambda x: x.replace(',',' '))

#split all the words in my text
df['text']= df['text'].str.split()

df.to_csv('unsmile_split.csv')
```

As always begin by importing pandas.
Then open the dataframe, and drop any rows that have a “nan” value.
I then simply do what I did above in part 3 to clean the data. But this time, I saved the dataframe as a file called “name_split.csv”.

## Our second to final step is to create a frequency table. As for why we want to do this, [refer to](http://dataaspirant.com/2017/02/06/naive-bayes-classifier-machine-learning/)
So to begin, we import the libraries we will use and the files we want. Here I import a module called sklearn because that library will help us split our dataframe into a test and train set.

```bash
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split

#Read the files with the tweets
fun = pd.read_csv('fun_split.csv')
happy = pd.read_csv('happy_split.csv')
unsmile = pd.read_csv('unsmile_split.csv')
sad = pd.read_csv('sad_split.csv')

#Get the wordbag
wordbag = pd.read_csv('wordbag.csv')
wordbag = wordbag.drop_duplicates()

#Classify tweets that had the word fun or happy in them as 1 (positive)
#and the others as 0 (negative)
fun['type'] = 1
happy['type'] = 1
unsmile['type'] = 0
sad['type'] = 0
   
#Join all of the dataframes into one big one for easier manipulation of a test/train split
df = pd.concat([happy,sad,unsmile,fun]).reset_index(drop=True)

#Create a test and train set by using the sklearn function train_test_split
train, test = train_test_split(df, test_size=0.2)

#Seperate the train data into a positive and negative set
train_positive = train[train['type'] ==1]
train_negative = train[train['type'] ==0]

positive_instance = len(train_positive)
negative_instance = len(train_negative)
print(positive_instance)
print(negative_instance)

#Create your frequency table
frequency['word'] = wordbag['word']
```

We also import the list of words we are interested in.
Lines 16–19 were used to convert the “fun” and “happy” classifiers into a positive mood, and the “unsmile” and “sad” classifier into a negative mood.
I then added all of the data into one big dataframe so I could create a train/test split on line 25.
From the training dataset, I then get my positive instances and negative instances.
I write line 35 so that I can have a final 2nd to last dataframe I wanted to work with.
At this point, we have a training set that has both positive and negative examples.
Our next goal is to go through the word bank and see how many of the positive instances have a given word in it. We repeat this for the negative instances and do this for all of the words in our word bank:

```bash
#Create your frequency table
frequency['word'] = wordbag['word']

word_bank = [0]*len(frequency)
positive = [0]*len(frequency)
negative = [0]*len(frequency)

#Go over all the words in the frequency table
for i in range(len(frequency)):
    
    #Get the word in the frequency table at a given row
    word = frequency['word'].iloc[i]
    word_bank[i] = word
             
    #Convert the word and attached single colons ont oboth sides of the word
    check = str("'") + word + str("'")
    
    #Count the number of instances that have the word at least once
    count = 0  
    
    #this iterates through each of the tweets in the positive train set
    for j in range(len(train_positive)):
        #This checks to see the number of time the said word appears in a given tweet
        appears = train_positive['text'].iloc[j].count(check)
        
        #If the word appears at least once, we count it as that tweet having it
        #We sum over all the tweets that the word appears at least once in 
        if appears > 0:
            count = count + 1
    positive[i] = count
            
    #Does the same thing but for negative numbers
    count = 0  
    for k in range(len(train_negative)):
        appears = train_negative['text'].iloc[k].count(check)
        if appears > 0:
            count = count + 1
    negative[i] = count
    print(i)

d = {'word': word_bank, 'positive': positive, 'negative': negative}
ftable = pd.DataFrame(data = d)

ftable.to_csv('ftable.csv')
```

To create the frequency table, I iterate through all the words in the word bank, and store the words into another array (I do this so I can create my final dataframe). I then go through my positive training set and iterate through all of the tweets in it.
Within each tweet, I see how many times the word I was looking for appears in it. If it is greater than 1, I add 1 to my counter.
This allows me to see how many tweets had at least 1 count of the word I wanted in it.
I then store this value into an array called “positive.” This way I have an array that tells me how many tweets had a given word in it.
I repeat this for the negative training set, an end the program by creating a dataframe.

## Step 5 (Testing our Naïve Bayes table):
The last step is to test how well our naïve bayes table does. While I could test it using the training set, I plan on writing a couple of sentences and seeing how it classifies my sentence and see how it does.
As always, I begin by importing pandas and numpy and the table I will be using.
Afterwards I get rid of any key words I think will appear, just in case my program was not perfect.
Test is the example sentence I am planning on testing.
Positive and negative instance are the number of entries I have for each pile (I had to manually get this number from the previous program)

```bash
import pandas as pd
import numpy as np

ftable = pd.read_csv('ftable.csv')
ftable = ftable[ftable['word'] != 'sad']
ftable = ftable[ftable['word'] != 'sad,']
ftable = ftable[ftable['word'] != ':(']
ftable = ftable[ftable['word'] != 'fun']
ftable = ftable[ftable['word'] != 'happy,']
ftable = ftable[ftable['word'] != 'happy']
ftable = ftable.drop_duplicates(subset = 'word')

test = 'i dont know what to do anymore'
positive_instance = 24070.0
negative_instance = 23930.0

#split all the words in my text
test_words = test.split()

prob_positive = float(positive_instance/(positive_instance+negative_instance))
prob_negative = 1 - prob_positive

pos_word = 1.0*prob_positive
neg_word = 1.0*prob_negative
for i in range(len(test_words)):
    word = test_words[i]
    #print(word)
    index_val = ftable.index[ftable['word'] == word]
    if (len(index_val) > 0):
        #print(index_val[0])
        pos_val = ftable['positive'].iloc[index_val[0]]
        neg_val = ftable['negative'].iloc[index_val[0]]
        pos_word = pos_word * pos_val/positive_instance
        neg_word = neg_word * neg_val/negative_instance
        
if pos_word > neg_word:
    print("The sentence was POSITIVE, with a probability of")
    print(pos_word/(pos_word+neg_word))
else:
    print("The sentence was NEGATIVE, with a probability of")
    print(neg_word/(pos_word+neg_word))
```

I then split the test sentence into its each individual words. Then I went back to the probabilities I needed for which you will have to review how its done.
The for loop goes through each word in the test sentence and finds the number of instances it appears in the frequency table so I can then calculate my probabilities.
If you remember, the probabilities I want to compare are:
For happy case: probability of having a happy instance * probability of word 1 being happy given that the sentence is happy * probability of word 2 being happy given that the sentence is happy * probability of word n being happy given that the sentence is happy
For the sad case, I repeat the above, but instead do it replace happy with sad.