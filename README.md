# Project: Wrangling and Analyze Data
## Data Gathering
In the cell below, gather **all** three pieces of data for this project and load them in the notebook. **Note:** the methods required to gather each data are different.
1. Directly download the WeRateDogs Twitter archive data (twitter_archive_enhanced.csv)
# Import required packages
import pandas as pd
import numpy as np
import requests
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns; sns.set()
import os
import json
from sqlalchemy import create_engine
from PIL import Image
from io import BytesIO
from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator
df1=pd.read_csv('twitter_archive_enhanced.csv')
df1.head(2)
df1.tweet_id=df1.expanded_urls.str.extract('(\d{18})')
df1.tweet_id[0:5]
2. Use the Requests library to download the tweet image prediction (image_predictions.tsv)
response= requests.get('https://d17h27t6h515a5.cloudfront.net/topher/2017/August/599fd2ad_image-predictions/image-predictions.tsv')
with open('image_prediction.tsv',mode='wb')as file:
    file.write(response.content)
df2=pd.read_csv('image_prediction.tsv',delimiter='\t')
3. Use the Tweepy library to query additional data via the Twitter API (tweet_json.txt)
import tweepy
from tweepy import OAuthHandler
import json
from timeit import default_timer as timer

# Query Twitter API for each tweet in the Twitter archive and save JSON in a text file
# These are hidden to comply with Twitter's API terms and conditions
consumer_key = 'HIDDEN'
consumer_secret = 'HIDDEN'
access_token = 'HIDDEN'
access_secret = 'HIDDEN'

auth = OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_secret)

api = tweepy.API(auth, wait_on_rate_limit=True)

# NOTE TO STUDENT WITH MOBILE VERIFICATION ISSUES:
# df_1 is a DataFrame with the twitter_archive_enhanced.csv file. You may have to
# change line 17 to match the name of your DataFrame with twitter_archive_enhanced.csv
# NOTE TO REVIEWER: this student had mobile verification issues so the following
# Twitter API code was sent to this student from a Udacity instructor
# Tweet IDs for which to gather additional data via Twitter's API
tweet_ids = df1.tweet_id.values
len(tweet_ids)
'''
# Query Twitter's API for JSON data for each tweet ID in the Twitter archive
count = 0
fails_dict = {}
start = timer()
# Save each tweet's returned JSON as a new line in a .txt file
with open('tweet-json.txt', 'w') as outfile:
    # This loop will likely take 20-30 minutes to run because of Twitter's rate limit
    for tweet_id in tweet_ids:
        count += 1
        print(str(count) + ": " + str(tweet_id))
        try:
            tweet = api.get_status(tweet_id, tweet_mode='extended')
            print("Success")
            json.dump(tweet._json, outfile)
            outfile.write('\n')
        except tweepy.TweepError as e:
            print("Fail")
            fails_dict[tweet_id] = e
            pass
end = timer()
print(end - start)
print(fails_dict)
'''
data = []
with open('tweet_json.txt') as f:    
        for line in f:         
             data.append(json.loads(line))
df3 = pd.DataFrame(data)
df3.to_csv(r'tweets_json.csv',index=False,header=True)
df3 = pd.read_csv('tweets_json.csv')

# Creating a dataframe from the previous List that contains the id, retweet count, and favorite count
tweet_api=pd.DataFrame(data, columns=['id', 'retweet_count', 'favorite_count'])
tweet_api.head()
# Changing the name of the id column to tweet_id
tweet_api = tweet_api.rename (columns = {'id' : 'tweet_id' })
tweet_api.head()
# Saving the dataframe to a csv file for future use (without the index column so it will not appear as unnamed column in the file)
tweet_api.to_csv('tweet_api.csv',index=False)
#Checking the file was saved correctly
x = pd.read_csv('tweet_api.csv')
x.head()
## Assessing Data
In this section, detect and document at least **eight (8) quality issues and two (2) tidiness issue**. You must use **both** visual assessment
programmatic assessement to assess the data.

**Note:** pay attention to the following key points when you access the data.

* You only want original ratings (no retweets) that have images. Though there are 5000+ tweets in the dataset, not all are dog ratings and some are retweets.
* Assessing and cleaning the entire dataset completely would require a lot of time, and is not necessary to practice and demonstrate your skills in data wrangling. Therefore, the requirements of this project are only to assess and clean at least 8 quality issues and at least 2 tidiness issues in this dataset.
* The fact that the rating numerators are greater than the denominators does not need to be cleaned. This [unique rating system](http://knowyourmeme.com/memes/theyre-good-dogs-brent) is a big part of the popularity of WeRateDogs.
* You do not need to gather the tweets beyond August 1st, 2017. You can, but note that you won't be able to gather the image predictions for these tweets since you don't have access to the algorithm used.


### Visual assessment
# Assessing twitter dataset visually.
# Displayinh first five rows.
df1.head()

df1.info()
df1
# Asscessing twitter dataset visually.
# Displaying last five rows.
df1.tail()

# Asscessing twitter dataset visually.
# Displaying random 20 rows.
df1.sample(20)
df1.rating_denominator.value_counts()
df2
df2.info()
df2
# Asscessing predictions dataset visually.
# Displaying first fifty rows
df2.head(50)
# Asscessing predictions dataset visually.
# Displaying last five rows.
df2.tail()
# Asscessing predictions dataset visually.
# Displaying 10 random rows.
df2.sample(10)
# Just curious how this photo was identified as a bathtub
df2.loc[1312, 'jpg_url']
from IPython.display import Image
Image(url= 'https://pbs.twimg.com/ext_tw_video_thumb/754481405627957248/pu/img/YY1eBDOlP9QFC4Bj.jpg')
# Now I understand :)
tweet_api.info()
# Asscessing tweet_api dataset visually.
# Displaying first five rows
tweet_api.head()
# Asscessing tweet_api dataset visually.
# Displaying last five rows.
df3.tail()
# Asscessing tweet_api dataset visually.
# displaying 10 random rows.
tweet_api.sample(10)
tweet_api
### Quality issues



#### A) Enhanced Twitter Archive
* Q1. There are 181 retweetes as indicated by retweeted _status_id.
* Q2. Some dog names are invalid (None, a, an, & the instead of name).
* Q3. Invalid tweet_id data type (integer instead of string).
* Q4. Invalid timestamp data type (string not datetime).
* Q5. 440 rating numerators less than 10 (ex: 1998).
* Q6. Row 313 has 0 denominator.
* Q7. 23 rating denominators not equal 10.

#### B) Tweet Image Predictions
* Q1. Missing photos for some IDs (2075 rows instead of 2356).
* Q2. Underscores are used in multi-word names in columns p1, p2, & p3 instead of spaces.
* Q3. Some P names start with an uppercase letter while others start with lowercase.

#### C) Tweet Data From Twitter API
* Q1. Missing entries (Only 2354 entries instead of 2356).
### Tidiness issues
* T1. Dog stage data is separated into 4 columns.

* T2. All data is related but divided into 3 separate dataframes.
## Cleaning Data
In this section, clean **all** of the issues you documented while assessing. 

**Note:** Make a copy of the original data before cleaning. Cleaning includes merging individual pieces of data according to the rules of [tidy data](https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html). The result should be a high-quality and tidy master pandas DataFrame (or DataFrames, if appropriate).
# Make copies of original pieces of data

df1_clean=df1.copy()
df2_clean=df2.copy()
df3_clean=tweet_api.copy()
df1_clean.head(1)
df2_clean.head(1)
df3_clean.head(1)
### Cleaning Tidiness Issues 
#### T1. Dog stage data is separated into 4 columns.

#### Define: 

Merge the 4 columnc into 1, called dog_stage
#### Code
# Extract dog stage from text column into the new dog_stage column
df1_clean['dog_stage'] = df1_clean['text'].str.extract('(doggo|floofer|pupper|puppo)')
df1_clean.head()
# Drop unrequired columns
df1_clean= df1_clean.drop(columns=['doggo','floofer','pupper', 'puppo'])
import re 

regex = r'''([+-]?([0-9]+[.])?[0-9]+\/[+-]?([0-9]+[.])?[0-9]+)'''

          #[+-]?([0-9]*[.])?[0-9]+\/[+-]?([0-9]*[.])?[0-9]+
def get_pattern(pat):
  try:
      return re.findall(regex, pat)[0][0]
  except Exception as e:
      return ''


df1['pattern'] = df1['text'].apply(get_pattern)
df1['fraction'] = df1['rating_numerator'].astype(str) + '/' + df1['rating_denominator'].astype(str)
df1[df1['pattern'] != df1['fraction']][['pattern', 'fraction']]
#### Test
df1_clean.dog_stage.value_counts()
#### T2. All data is related but divided into 3 separate dataframes.
#### Define:

Merge all dataframes into 1 based on tweet_id

#### Code
# converting tweet_id datatype to int in order to merge

df1_clean['tweet_id'] = df1_clean['tweet_id'].fillna(0)

df1_clean['tweet_id'] = df1_clean['tweet_id'].astype(int)


# Merging the cleaned Enhanced Twitter Archive data with the data from Twitter API
df1_clean= pd.merge(df1_clean, df3_clean, on='tweet_id', how= 'left')

# Merging the resulting merged archive with the Tweet Image Predictions
df1_clean = pd.merge(df1_clean, df2_clean , on='tweet_id', how='left')
#### Test
df1_clean.info()
### Cleaning Quality Issues
Not all quality issues will be cleaned since such data will not be used for analysis
### AQ1. There are 181 retweetes as indicated by retweeted_status_id
#### Define: 

Delete rows that represent retweets and all related columns

#### Code
#Keep only original tweets that have no retweet statud id
df1_clean=df1_clean[df1_clean.retweeted_status_id.isnull()]
df1_clean.info()
# Delete related colums
df1_clean= df1_clean.drop(columns=['retweeted_status_id', 'retweeted_status_user_id', 'retweeted_status_timestamp'])
#### Test

df1_clean.info()
### AQ2. Some dog names are invalid (None, a, an, & the instead of name)
#### Define: 

Convert invalid names (None or starting with lower case letters) to NaN and extract the correct names from the text column (after the word "named")


#### Code
df1_clean.name = df1_clean.name.replace(regex=['^[a-z]+', 'None'], value= np.nan)

# Checking number of null values in name column after conversion
sum(df1_clean.name.isnull())
# Declare a function to extract names from text column, and return NaN if there is no "named" word

def function(text):
    txt_list = text.split()
    for word in txt_list:
        if word.lower() == 'named':
            name_index = txt_list.index(word) + 1 # word after "named" 
            return txt_list[name_index]
        else:
            pass
    return np.nan
#np.where(condition, what to do if condition is true, what to do if condition is false)
df1_clean.name = np.where(df1_clean.name.isnull(), df1_clean.text.apply(function), df1_clean.name)
#### Test

sum(df1_clean.name.isnull())
# Names were added in place of some null values.
### AQ3. Invalid tweet_id data type (integer instead of string)
#### Define: 

Correct invalid data type by converting tweet_id to string

#### Code
# Convert tweet_id to string since no operation will be perforned on its values
df1_clean.tweet_id= df1_clean.tweet_id.astype(str)
#### Test

df1_clean.info()
### AQ4. Invalid timestamp data type (string not datetime)
#### Define: 

Correct invalid data type by converting timestamp to datetime

#### Code
# Convert timestampm to datetime
df1_clean.timestamp=pd.to_datetime(df1_clean.timestamp)
#### Test

df1_clean.info()
### BQ1. Missing photos for some IDs (2075 rows instead of 2356).
#### Define: 

Delete rows with missing photos

#### Code
df2_clean = df2_clean[df2_clean.jpg_url.notnull()]
#### Test

df2_clean.info()
### BQ2. Underscores are used in multi-word names in columns p1, p2, & p3 instead of spaces.
#### Define: 

Replace underscores in multi-word names in columns p1, p2, & p3 with spaces.
#### Code
df2_clean.p1 = df2_clean.p1.str.replace('_',' ')
df2_clean.p2 = df2_clean.p2.str.replace('_',' ')
df2_clean.p3 = df2_clean.p3.str.replace('_',' ')
#### Test

df2_clean.p1.head(10)
df2_clean.p2.head(10)
df2_clean.p3.head(10)
### BQ3. Some P names start with an uppercase letter while others start with lowercase.
#### Define: 

Convert lowercase letters to uppercase
#### Code
df2_clean.p1 = df2_clean.p1.str.title()
df2_clean.p2 = df2_clean.p2.str.title()
df2_clean.p3 = df2_clean.p3.str.title()
#### Test

df2_clean.p1.head(10)
df2_clean.p2.head(10)
df2_clean.p3.head(10)
### CQ1. Missing entries (Only 2354 entries instead of 2356).
#### Define: 

Delete rows without retweet count entries
#### Code
Deleted in previus steps while cleaning other issues
#### Test

sum(df3_clean.retweet_count.isnull())
### Final Test
df1_clean.info()

df2_clean.info()

df3_clean.info()

## Storing Data
Save gathered, assessed, and cleaned master dataset to a CSV file named "twitter_archive_master.csv".
df1_clean.to_csv('twitter_archive_master.csv')
## Analyzing and Visualizing Data
In this section, analyze and visualize your wrangled data. You must produce at least **three (3) insights and one (1) visualization.**
### A. The percentage of different dog stages
stage_df = df1_clean.dog_stage.value_counts()
stage_df
# Creating a pie chert

plt.pie(stage_df,
      labels= ['Pupper','Doggo', 'Puppo', 'Floofer'], 
      autopct= '%1.1f%%', # To show percent on plot. 1.1 formats the percentage to the tenth place.
      shadow=True,
      explode=(0.1, 0.2, 0.2, 0.3))
plt.title('Percentage of Dogs Stages')
plt.axis('equal'); # By default, matplotlib creates pie charts with a tilt. This Line remove this tilt
### Insights:
1. Pupper has the highest percentage.

2. Floofer has the lowest percentage.

### B. Relationship between retweet count and favorite count
plt.scatter(df3_clean.retweet_count, df3_clean.favorite_count)
plt.title('Relationship between retweet count and favorite count') 
plt.xlabel ('Retweet Count')
plt.ylabel('Favorite Count');
### Insights:

As shown in the graph, there is a positive correlation relationship in such as Retweet Count increases, Favorite Count increases as well.
