
# Data Wrangling

by Alina Bolat


```python
# Libraries
import pandas as pd
import numpy as np
import datetime as dt
import requests
import tweepy
import json
import time
import re
import os
# Set the maximum width of columns to display tweet text in full
pd.set_option('display.max_colwidth', -1)
```

## Gather
Gathering process consists of following three data sets:  
1. **twitter_archive_enhanced.csv** is avalable for manual download.
2. **image_predictions.tsv** is avalable through a link for programmatic download from the Udacity Servers.
3. **tweet_json.txt** is to be scraped using Twitter API - tweepy.
***
### Twitter Archive


```python
# Import the csv file and store it in a dataframe
twitter_archive = pd.read_csv('twitter-archive-enhanced.csv', encoding = 'utf-8')
# Check the outcome
twitter_archive.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2356 entries, 0 to 2355
    Data columns (total 17 columns):
    tweet_id                      2356 non-null int64
    in_reply_to_status_id         78 non-null float64
    in_reply_to_user_id           78 non-null float64
    timestamp                     2356 non-null object
    source                        2356 non-null object
    text                          2356 non-null object
    retweeted_status_id           181 non-null float64
    retweeted_status_user_id      181 non-null float64
    retweeted_status_timestamp    181 non-null object
    expanded_urls                 2297 non-null object
    rating_numerator              2356 non-null int64
    rating_denominator            2356 non-null int64
    name                          2356 non-null object
    doggo                         2356 non-null object
    floofer                       2356 non-null object
    pupper                        2356 non-null object
    puppo                         2356 non-null object
    dtypes: float64(4), int64(3), object(10)
    memory usage: 313.0+ KB
    

***
### Image Predictions


```python
###
# # Using Requests library download a file
# url = 'https://d17h27t6h515a5.cloudfront.net/topher/2017/August/599fd2ad_image-predictions/image-predictions.tsv'
# response = requests.get(url)
# # Save the downloaded file
# with open(url.split('/')[-1], mode = 'wb') as outfile:
#     outfile.write(response.content)
###
```


```python
# Import the csv file and store it in a dataframe
image_predictions = pd.read_csv('image-predictions.tsv', sep = '\t', encoding = 'utf-8')
# Check the result
image_predictions.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2075 entries, 0 to 2074
    Data columns (total 12 columns):
    tweet_id    2075 non-null int64
    jpg_url     2075 non-null object
    img_num     2075 non-null int64
    p1          2075 non-null object
    p1_conf     2075 non-null float64
    p1_dog      2075 non-null bool
    p2          2075 non-null object
    p2_conf     2075 non-null float64
    p2_dog      2075 non-null bool
    p3          2075 non-null object
    p3_conf     2075 non-null float64
    p3_dog      2075 non-null bool
    dtypes: bool(3), float64(3), int64(2), object(4)
    memory usage: 152.1+ KB
    

***
### Twitter Data


```python
###
# # Twitter API Authorisation - CONFIDENTIAL INFORMATION REMOVED
# consumer_key = ''
# consumer_secret = ''
# access_token = ''
# access_secret = ''
# 
# auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
# auth.set_access_token(access_token, access_secret)
# 
# api = tweepy.API(auth, wait_on_rate_limit = True, wait_on_rate_limit_notify = True)
###
```


```python
###
# # Calculate the time of excution
# start = time.time()
# # A separate list for catching errors
# errors = []
# 
# with open('tweet_json.txt', 'w') as file:
#     for tweet_id in twitter_archive['tweet_id']:
#         try:
#             tweet = api.get_status(tweet_id, tweet_mode = 'extended') # For every tweet_id in the list
#             file.write(json.dumps(tweet._json) + '\n') # write full JSON status and create a new line
#         except Exception as e:
#             print(str(tweet_id) + " " + str(e)) # Print the missing Tweet ID and the error text
#             errors.append(tweet_id)
# 
# # Calculate the time of excution
# end = time.time()
# print(end - start)
###
```


```python
###
# # List of errors
# errors_df = pd.DataFrame(twitter_archive.loc[twitter_archive['tweet_id'].isin(errors),:])
# # Save the list of errors for Assessment
# errors_df.to_csv('tweet_json_errors.csv', index=False, encoding = 'utf-8')
###
```


```python
# Lists of variables of interest
tweet_id = []
favorite_count = []
retweet_count = []
with open('tweet_json.txt', mode = 'r') as f:
     for line in f.readlines():
            tweet_json = json.loads(line)
            tweet_id.append(tweet_json['id'])
            favorite_count.append(tweet_json['favorite_count']) 
            retweet_count.append(tweet_json['retweet_count'])
            
# Store each variable in an identically named columns in a new dataframe           
tweet_json = pd.DataFrame({'tweet_id' : tweet_id, 
                           'favorite_count' : favorite_count, 
                           'retweet_count' : retweet_count})
# Check outcome
tweet_json.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2345 entries, 0 to 2344
    Data columns (total 3 columns):
    favorite_count    2345 non-null int64
    retweet_count     2345 non-null int64
    tweet_id          2345 non-null int64
    dtypes: int64(3)
    memory usage: 55.0 KB
    

***
## Assess

**Twitter archive** is a dataframe with 17 Columns and 2356 observations. Straight away it is obvious that there are missing values and other flaws in the dataset that will need to be addressed.  
**Image Predictions** dataframe consists of 12 columns and 2075 observations. There appear to be no missing values, but further investigation is required to determine the importance of columns.  
**Twitter Json** is a dataframe programmaticaly gathered from the twitter servers, it has 2345 observations and 3 variables, it will serve as an additional data for the final dataframe.  

In this part of the project the datasets will be assessed Quality and Tidiness. In order to narrow down data wrangling scope, it always helps me to pencil down areas of interest for the EDA and ask preli/minary questions. These questions will be changed and refined as the process goes on.  

* Is the rating subjective or dependant on certain other traits?
* Which dog stage is posted most and/or earns highest scores? 
* Which dog breed is the most popular?
* Which dog breed is the most liked/has highest scores?
* What are main sources of tweets for We Rate Dogs account?
* What are the most viral tweets in the sample (most retweets and quotes)?
* Are there any trends in time or season when retweets/favourites are more prevalent?


```python
# Visual assessment of the top and the bottom of the dataset, 
# I like to print out the dataset whole, as Jupyter Notebooks makes it very easy 
# to see the Head and the Tail of a dataset by abreviating the dataframe list.
twitter_archive
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 16:23:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Phineas. He's a mystical boy. Only ever appears in the hole of a donut. 13/10 https://t.co/MgUWQ76dJU</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892420643555336193/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 00:17:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Tilly. She's just checking pup on you. Hopes you're doing ok. If not, she's available for pats, snugs, boops, the whole bit. 13/10 https://t.co/0Xxu71qeIV</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892177421306343426/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-31 00:18:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Archie. He is a rare Norwegian Pouncing Corgo. Lives in the tall grass. You never know when one may strike. 12/10 https://t.co/wUnZnhtVJB</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891815181378084864/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-30 15:58:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Darla. She commenced a snooze mid meal. 13/10 happens to the best of us https://t.co/tD36da7qLQ</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891689557279858688/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 16:00:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Franklin. He would like you to stop calling him "cute." He is a very fierce shark and should be respected as such. 12/10 #BarkWeek https://t.co/AtUZn91f7f</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891327558926688256/photo/1,https://twitter.com/dog_rates/status/891327558926688256/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>5</th>
      <td>891087950875897856</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 00:08:17 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a majestic great white breaching off South Africa's coast. Absolutely h*ckin breathtaking. 13/10 (IG: tucker_marlo) #BarkWeek https://t.co/kQ04fDDRmh</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891087950875897856/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>6</th>
      <td>890971913173991426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-28 16:27:12 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Meet Jax. He enjoys ice cream so much he gets nervous around it. 13/10 help Jax enjoy more things by clicking below\n\nhttps://t.co/Zr4hWfAs1H https://t.co/tVJBRMnhxl</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://gofundme.com/ydvmve-surgery-for-jax,https://twitter.com/dog_rates/status/890971913173991426/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Jax</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>7</th>
      <td>890729181411237888</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-28 00:22:40 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>When you watch your owner call another dog a good boy but then they turn back to you and say you're a great boy. 13/10 https://t.co/v0nONBcwxq</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/890729181411237888/photo/1,https://twitter.com/dog_rates/status/890729181411237888/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>8</th>
      <td>890609185150312448</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-27 16:25:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Zoey. She doesn't want to be one of the scary sharks. Just wants to be a snuggly pettable boatpet. 13/10 #BarkWeek https://t.co/9TwLuAGH0b</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/890609185150312448/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Zoey</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>9</th>
      <td>890240255349198849</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-26 15:59:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Cassie. She is a college pup. Studying international doggo communication and stick theory. 14/10 so elegant much sophisticate https://t.co/t1bfwz5S2A</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/890240255349198849/photo/1</td>
      <td>14</td>
      <td>10</td>
      <td>Cassie</td>
      <td>doggo</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>10</th>
      <td>890006608113172480</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-26 00:31:25 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Koda. He is a South Australian deckshark. Deceptively deadly. Frighteningly majestic. 13/10 would risk a petting #BarkWeek https://t.co/dVPW0B0Mme</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/890006608113172480/photo/1,https://twitter.com/dog_rates/status/890006608113172480/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Koda</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>11</th>
      <td>889880896479866881</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-25 16:11:53 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Bruno. He is a service shark. Only gets out of the water to assist you. 13/10 terrifyingly good boy https://t.co/u1XPQMl29g</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/889880896479866881/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Bruno</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>12</th>
      <td>889665388333682689</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-25 01:55:32 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here's a puppo that seems to be on the fence about something haha no but seriously someone help her. 13/10 https://t.co/BxvuXk0UCm</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/889665388333682689/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>puppo</td>
    </tr>
    <tr>
      <th>13</th>
      <td>889638837579907072</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-25 00:10:02 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Ted. He does his best. Sometimes that's not enough. But it's ok. 12/10 would assist https://t.co/f8dEDcrKSR</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/889638837579907072/photo/1,https://twitter.com/dog_rates/status/889638837579907072/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Ted</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>14</th>
      <td>889531135344209921</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-24 17:02:04 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Stuart. He's sporting his favorite fanny pack. Secretly filled with bones only. 13/10 puppared puppo #BarkWeek https://t.co/y70o6h3isq</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/889531135344209921/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Stuart</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>puppo</td>
    </tr>
    <tr>
      <th>15</th>
      <td>889278841981685760</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-24 00:19:32 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Oliver. You're witnessing one of his many brutal attacks. Seems to be playing with his victim. 13/10 fr*ckin frightening #BarkWeek https://t.co/WpHvrQedPb</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/889278841981685760/video/1</td>
      <td>13</td>
      <td>10</td>
      <td>Oliver</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>16</th>
      <td>888917238123831296</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-23 00:22:39 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Jim. He found a fren. Taught him how to sit like the good boys. 12/10 for both https://t.co/chxruIOUJN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/888917238123831296/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Jim</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>17</th>
      <td>888804989199671297</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-22 16:56:37 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Zeke. He has a new stick. Very proud of it. Would like you to throw it for him without taking it. 13/10 would do my best https://t.co/HTQ77yNQ5K</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/888804989199671297/photo/1,https://twitter.com/dog_rates/status/888804989199671297/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Zeke</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>18</th>
      <td>888554962724278272</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-22 00:23:06 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Ralphus. He's powering up. Attempting maximum borkdrive. 13/10 inspirational af https://t.co/YnYAFCTTiK</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/888554962724278272/photo/1,https://twitter.com/dog_rates/status/888554962724278272/photo/1,https://twitter.com/dog_rates/status/888554962724278272/photo/1,https://twitter.com/dog_rates/status/888554962724278272/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Ralphus</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>19</th>
      <td>888202515573088257</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-21 01:02:36 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: This is Canela. She attempted some fancy porch pics. They were unsuccessful. 13/10 someone help her https://t.co/cLyzpcUcMX</td>
      <td>8.874740e+17</td>
      <td>4.196984e+09</td>
      <td>2017-07-19 00:47:34 +0000</td>
      <td>https://twitter.com/dog_rates/status/887473957103951883/photo/1,https://twitter.com/dog_rates/status/887473957103951883/photo/1,https://twitter.com/dog_rates/status/887473957103951883/photo/1,https://twitter.com/dog_rates/status/887473957103951883/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Canela</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>20</th>
      <td>888078434458587136</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-20 16:49:33 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Gerald. He was just told he didn't get the job he interviewed for. A h*ckin injustice. 12/10 didn't want the job anyway https://t.co/DK7iDPfuRX</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/888078434458587136/photo/1,https://twitter.com/dog_rates/status/888078434458587136/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Gerald</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>21</th>
      <td>887705289381826560</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-19 16:06:48 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Jeffrey. He has a monopoly on the pool noodles. Currently running a 'boop for two' midweek sale. 13/10 h*ckin strategic https://t.co/PhrUk20Q64</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/887705289381826560/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Jeffrey</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>22</th>
      <td>887517139158093824</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-19 03:39:09 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>I've yet to rate a Venezuelan Hover Wiener. This is such an honor. 14/10 paw-inspiring af (IG: roxy.thedoxy) https://t.co/20VrLAA8ba</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/887517139158093824/video/1</td>
      <td>14</td>
      <td>10</td>
      <td>such</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>23</th>
      <td>887473957103951883</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-19 00:47:34 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Canela. She attempted some fancy porch pics. They were unsuccessful. 13/10 someone help her https://t.co/cLyzpcUcMX</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/887473957103951883/photo/1,https://twitter.com/dog_rates/status/887473957103951883/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Canela</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>24</th>
      <td>887343217045368832</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-18 16:08:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>You may not have known you needed to see this today. 13/10 please enjoy (IG: emmylouroo) https://t.co/WZqNqygEyV</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/887343217045368832/video/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>25</th>
      <td>887101392804085760</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-18 00:07:08 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This... is a Jubilant Antarctic House Bear. We only rate dogs. Please only send dogs. Thank you... 12/10 would suffocate in floof https://t.co/4Ad1jzJSdp</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/887101392804085760/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>26</th>
      <td>886983233522544640</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-17 16:17:36 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Maya. She's very shy. Rarely leaves her cup. 13/10 would find her an environment to thrive in https://t.co/I6oNy0CgiT</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/886983233522544640/photo/1,https://twitter.com/dog_rates/status/886983233522544640/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Maya</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>27</th>
      <td>886736880519319552</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-16 23:58:41 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Mingus. He's a wonderful father to his smol pup. Confirmed 13/10, but he needs your help\n\nhttps://t.co/bVi0Yr4Cff https://t.co/ISvKOSkd5b</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://www.gofundme.com/mingusneedsus,https://twitter.com/dog_rates/status/886736880519319552/photo/1,https://twitter.com/dog_rates/status/886736880519319552/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Mingus</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>28</th>
      <td>886680336477933568</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-16 20:14:00 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Derek. He's late for a dog meeting. 13/10 pet...al to the metal https://t.co/BCoWue0abA</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/886680336477933568/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Derek</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>29</th>
      <td>886366144734445568</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-15 23:25:31 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Roscoe. Another pupper fallen victim to spontaneous tongue ejections. Get the BlepiPen immediate. 12/10 deep breaths Roscoe https://t.co/RGE08MIJox</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/886366144734445568/photo/1,https://twitter.com/dog_rates/status/886366144734445568/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Roscoe</td>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2326</th>
      <td>666411507551481857</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-17 00:24:19 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is quite the dog. Gets really excited when not in water. Not very soft tho. Bad at fetch. Can't do tricks. 2/10 https://t.co/aMCTNWO94t</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666411507551481857/photo/1</td>
      <td>2</td>
      <td>10</td>
      <td>quite</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2327</th>
      <td>666407126856765440</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-17 00:06:54 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a southern Vesuvius bumblegruff. Can drive a truck (wow). Made friends with 5 other nifty dogs (neat). 7/10 https://t.co/LopTBkKa8h</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666407126856765440/photo/1</td>
      <td>7</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2328</th>
      <td>666396247373291520</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 23:23:41 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Oh goodness. A super rare northeast Qdoba kangaroo mix. Massive feet. No pouch (disappointing). Seems alert. 9/10 https://t.co/Dc7b0E8qFE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666396247373291520/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2329</th>
      <td>666373753744588802</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 21:54:18 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Those are sunglasses and a jean jacket. 11/10 dog cool af https://t.co/uHXrPkUEyl</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666373753744588802/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2330</th>
      <td>666362758909284353</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 21:10:36 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Unique dog here. Very small. Lives in container of Frosted Flakes (?). Short legs. Must be rare 6/10 would still pet https://t.co/XMD9CwjEnM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666362758909284353/photo/1</td>
      <td>6</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2331</th>
      <td>666353288456101888</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 20:32:58 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a mixed Asiago from the Galápagos Islands. Only one ear working. Big fan of marijuana carpet. 8/10 https://t.co/tltQ5w9aUO</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666353288456101888/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2332</th>
      <td>666345417576210432</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 20:01:42 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Look at this jokester thinking seat belt laws don't apply to him. Great tongue tho 10/10 https://t.co/VFKG1vxGjB</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666345417576210432/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2333</th>
      <td>666337882303524864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 19:31:45 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is an extremely rare horned Parthenon. Not amused. Wears shoes. Overall very nice. 9/10 would pet aggressively https://t.co/QpRjllzWAL</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666337882303524864/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2334</th>
      <td>666293911632134144</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 16:37:02 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a funny dog. Weird toes. Won't come down. Loves branch. Refuses to eat his food. Hard to cuddle with. 3/10 https://t.co/IIXis0zta0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666293911632134144/photo/1</td>
      <td>3</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2335</th>
      <td>666287406224695296</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 16:11:11 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is an Albanian 3 1/2 legged  Episcopalian. Loves well-polished hardwood flooring. Penis on the collar. 9/10 https://t.co/d9NcXFKwLv</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666287406224695296/photo/1</td>
      <td>1</td>
      <td>2</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2336</th>
      <td>666273097616637952</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 15:14:19 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Can take selfies 11/10 https://t.co/ws2AMaNwPW</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666273097616637952/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2337</th>
      <td>666268910803644416</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 14:57:41 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Very concerned about fellow dog trapped in computer. 10/10 https://t.co/0yxApIikpk</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666268910803644416/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2338</th>
      <td>666104133288665088</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 04:02:55 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Not familiar with this breed. No tail (weird). Only 2 legs. Doesn't bark. Surprisingly quick. Shits eggs. 1/10 https://t.co/Asgdc6kuLX</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666104133288665088/photo/1</td>
      <td>1</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2339</th>
      <td>666102155909144576</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 03:55:04 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Oh my. Here you are seeing an Adobe Setter giving birth to twins!!! The world is an amazing place. 11/10 https://t.co/11LvqN4WLq</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666102155909144576/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2340</th>
      <td>666099513787052032</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 03:44:34 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Can stand on stump for what seems like a while. Built that birdhouse? Impressive. Made friends with a squirrel. 8/10 https://t.co/Ri4nMTLq5C</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666099513787052032/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2341</th>
      <td>666094000022159362</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 03:22:39 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This appears to be a Mongolian Presbyterian mix. Very tired. Tongue slip confirmed. 9/10 would lie down with https://t.co/mnioXo3IfP</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666094000022159362/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2342</th>
      <td>666082916733198337</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 02:38:37 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a well-established sunblockerspaniel. Lost his other flip-flop. 6/10 not very waterproof https://t.co/3RU6x0vHB7</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666082916733198337/photo/1</td>
      <td>6</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2343</th>
      <td>666073100786774016</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 01:59:36 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Let's hope this flight isn't Malaysian (lol). What a dog! Almost completely camouflaged. 10/10 I trust this pilot https://t.co/Yk6GHE9tOY</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666073100786774016/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2344</th>
      <td>666071193221509120</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 01:52:02 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a northern speckled Rhododendron. Much sass. Gives 0 fucks. Good tongue. 9/10 would caress sensually https://t.co/ZoL8kq2XFx</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666071193221509120/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2345</th>
      <td>666063827256086533</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 01:22:45 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is the happiest dog you will ever see. Very committed owner. Nice couch. 10/10 https://t.co/RhUEAloehK</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666063827256086533/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>the</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2346</th>
      <td>666058600524156928</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 01:01:59 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is the Rand Paul of retrievers folks! He's probably good at poker. Can drink beer (lol rad). 8/10 good dog https://t.co/pYAJkAe76p</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666058600524156928/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>the</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2347</th>
      <td>666057090499244032</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:55:59 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>My oh my. This is a rare blond Canadian terrier on wheels. Only $8.98. Rather docile. 9/10 very rare https://t.co/yWBqbrzy8O</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666057090499244032/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2348</th>
      <td>666055525042405380</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:49:46 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is a Siberian heavily armored polar bear mix. Strong owner. 10/10 I would do unspeakable things to pet this dog https://t.co/rdivxLiqEt</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666055525042405380/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2349</th>
      <td>666051853826850816</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:35:11 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is an odd dog. Hard on the outside but loving on the inside. Petting still fun. Doesn't play catch well. 2/10 https://t.co/v5A4vzSDdc</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666051853826850816/photo/1</td>
      <td>2</td>
      <td>10</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2350</th>
      <td>666050758794694657</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:30:50 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a truly beautiful English Wilson Staff retriever. Has a nice phone. Privileged. 10/10 would trade lives with https://t.co/fvIbQfHjIe</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666050758794694657/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2351</th>
      <td>666049248165822465</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:24:50 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a 1949 1st generation vulpix. Enjoys sweat tea and Fox News. Cannot be phased. 5/10 https://t.co/4B7cOc1EDq</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666049248165822465/photo/1</td>
      <td>5</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2352</th>
      <td>666044226329800704</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:04:52 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a purebred Piers Morgan. Loves to Netflix and chill. Always looks like he forgot to unplug the iron. 6/10 https://t.co/DWnyCjf2mx</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666044226329800704/photo/1</td>
      <td>6</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2353</th>
      <td>666033412701032449</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-15 23:21:54 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is a very happy pup. Big fan of well-maintained decks. Just look at that tongue. 9/10 would cuddle af https://t.co/y671yMhoiR</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666033412701032449/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2354</th>
      <td>666029285002620928</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-15 23:05:30 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a western brown Mitsubishi terrier. Upset about leaf. Actually 2 dogs here. 7/10 would walk the shit out of https://t.co/r7mOb2m0UI</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666029285002620928/photo/1</td>
      <td>7</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2355</th>
      <td>666020888022790149</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-15 22:32:08 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a Japanese Irish Setter. Lost eye in Vietnam (?). Big fan of relaxing on stair. 8/10 would pet https://t.co/BLDqew2Ijj</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666020888022790149/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
<p>2356 rows × 17 columns</p>
</div>




```python
# Random sample of 5 twitter entries
twitter_archive.sample(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>509</th>
      <td>812466873996607488</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-12-24 01:16:12 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Mary. She's desperately trying to recreate her Coachella experience.   12/10 downright h*ckin adorable https://t.co/BAJrfPvtux</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/812466873996607488/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Mary</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1656</th>
      <td>683357973142474752</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-01-02 18:43:31 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>"Have a seat, son. There are some things we need to discuss" 10/10 https://t.co/g4G5tvfTVd</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/683357973142474752/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1187</th>
      <td>718460005985447936</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-04-08 15:26:28 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Meet Bowie. He's listening for underground squirrels. Smart af. Left eye is considerably magical. 9/10 would so pet https://t.co/JyNmyjy3fe</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/718460005985447936/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>Bowie</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1598</th>
      <td>686035780142297088</td>
      <td>6.860340e+17</td>
      <td>4.196984e+09</td>
      <td>2016-01-10 04:04:10 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Yes I do realize a rating of 4/20 would've been fitting. However, it would be unjust to give these cooperative pups that low of a rating</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4</td>
      <td>20</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2116</th>
      <td>670427002554466305</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-28 02:20:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Deciduous Trimester mix named Spork. Only 1 ear works. No seat belt. Incredibly reckless. 9/10 still cute https://t.co/CtuJoLHiDo</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/670427002554466305/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
# List sources
twitter_archive.source.value_counts()
```




    <a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>     2221
    <a href="http://vine.co" rel="nofollow">Vine - Make a Scene</a>                        91  
    <a href="http://twitter.com" rel="nofollow">Twitter Web Client</a>                     33  
    <a href="https://about.twitter.com/products/tweetdeck" rel="nofollow">TweetDeck</a>    11  
    Name: source, dtype: int64




```python
# There are multiple dogs in single tweet
twitter_archive.loc[531,'text']
```




    'Here we have Burke (pupper) and Dexter (doggo). Pupper wants to be exactly like doggo. Both 12/10 would pet at same time https://t.co/ANBpEYHaho'




```python
# List all the numerators
twitter_archive.rating_numerator.value_counts()
```




    12      558
    11      464
    10      461
    13      351
    9       158
    8       102
    7       55 
    14      54 
    5       37 
    6       32 
    3       19 
    4       17 
    1       9  
    2       9  
    420     2  
    0       2  
    15      2  
    75      2  
    80      1  
    20      1  
    24      1  
    26      1  
    44      1  
    50      1  
    60      1  
    165     1  
    84      1  
    88      1  
    144     1  
    182     1  
    143     1  
    666     1  
    960     1  
    1776    1  
    17      1  
    27      1  
    45      1  
    99      1  
    121     1  
    204     1  
    Name: rating_numerator, dtype: int64




```python
# There are very high ratings, might want to address this manualy, as it will skew the data
twitter_archive.loc[twitter_archive['rating_numerator'] == 1776,['tweet_id','text']]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>979</th>
      <td>749981277374128128</td>
      <td>This is Atticus. He's quite simply America af. 1776/10 https://t.co/GRXwMxLBkh</td>
    </tr>
  </tbody>
</table>
</div>




```python
# There is also Snoop Dog, who has to go
twitter_archive.loc[twitter_archive['rating_numerator'] == 420,['tweet_id','text']]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>188</th>
      <td>855862651834028034</td>
      <td>@dhmontgomery We also gave snoop dogg a 420/10 but I think that predated your research</td>
    </tr>
    <tr>
      <th>2074</th>
      <td>670842764863651840</td>
      <td>After so many requests... here you go.\n\nGood dogg. 420/10 https://t.co/yfAAo1gdeY</td>
    </tr>
  </tbody>
</table>
</div>




```python
# List all the denominators
twitter_archive.rating_denominator.value_counts()
```




    10     2333
    11     3   
    50     3   
    80     2   
    20     2   
    2      1   
    16     1   
    40     1   
    70     1   
    15     1   
    90     1   
    110    1   
    120    1   
    130    1   
    150    1   
    170    1   
    7      1   
    0      1   
    Name: rating_denominator, dtype: int64




```python
# View tweets which do not have rating denominator of 10
print('Total number of instances: ', len(twitter_archive.loc[twitter_archive.rating_denominator!=10,
                              ['tweet_id','text','rating_numerator','rating_denominator']]))
twitter_archive.loc[twitter_archive.rating_denominator!=10,
                              ['tweet_id','text','rating_numerator','rating_denominator']]
```

    Total number of instances:  23
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>text</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>313</th>
      <td>835246439529840640</td>
      <td>@jonnysun @Lin_Manuel ok jomny I know you're excited but 960/00 isn't a valid rating, 13/10 is tho</td>
      <td>960</td>
      <td>0</td>
    </tr>
    <tr>
      <th>342</th>
      <td>832088576586297345</td>
      <td>@docmisterio account started on 11/15/15</td>
      <td>11</td>
      <td>15</td>
    </tr>
    <tr>
      <th>433</th>
      <td>820690176645140481</td>
      <td>The floofs have been released I repeat the floofs have been released. 84/70 https://t.co/NIYC820tmd</td>
      <td>84</td>
      <td>70</td>
    </tr>
    <tr>
      <th>516</th>
      <td>810984652412424192</td>
      <td>Meet Sam. She smiles 24/7 &amp;amp; secretly aspires to be a reindeer. \nKeep Sam smiling by clicking and sharing this link:\nhttps://t.co/98tB8y7y7t https://t.co/LouL5vdvxx</td>
      <td>24</td>
      <td>7</td>
    </tr>
    <tr>
      <th>784</th>
      <td>775096608509886464</td>
      <td>RT @dog_rates: After so many requests, this is Bretagne. She was the last surviving 9/11 search dog, and our second ever 14/10. RIP https:/…</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>902</th>
      <td>758467244762497024</td>
      <td>Why does this never happen at my front door... 165/150 https://t.co/HmwrdfEfUE</td>
      <td>165</td>
      <td>150</td>
    </tr>
    <tr>
      <th>1068</th>
      <td>740373189193256964</td>
      <td>After so many requests, this is Bretagne. She was the last surviving 9/11 search dog, and our second ever 14/10. RIP https://t.co/XAVDNDaVgQ</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>1120</th>
      <td>731156023742988288</td>
      <td>Say hello to this unbelievably well behaved squad of doggos. 204/170 would try to pet all at once https://t.co/yGQI3He3xv</td>
      <td>204</td>
      <td>170</td>
    </tr>
    <tr>
      <th>1165</th>
      <td>722974582966214656</td>
      <td>Happy 4/20 from the squad! 13/10 for all https://t.co/eV1diwds8a</td>
      <td>4</td>
      <td>20</td>
    </tr>
    <tr>
      <th>1202</th>
      <td>716439118184652801</td>
      <td>This is Bluebert. He just saw that both #FinalFur match ups are split 50/50. Amazed af. 11/10 https://t.co/Kky1DPG4iq</td>
      <td>50</td>
      <td>50</td>
    </tr>
    <tr>
      <th>1228</th>
      <td>713900603437621249</td>
      <td>Happy Saturday here's 9 puppers on a bench. 99/90 good work everybody https://t.co/mpvaVxKmc1</td>
      <td>99</td>
      <td>90</td>
    </tr>
    <tr>
      <th>1254</th>
      <td>710658690886586372</td>
      <td>Here's a brigade of puppers. All look very prepared for whatever happens next. 80/80 https://t.co/0eb7R1Om12</td>
      <td>80</td>
      <td>80</td>
    </tr>
    <tr>
      <th>1274</th>
      <td>709198395643068416</td>
      <td>From left to right:\nCletus, Jerome, Alejandro, Burp, &amp;amp; Titson\nNone know where camera is. 45/50 would hug all at once https://t.co/sedre1ivTK</td>
      <td>45</td>
      <td>50</td>
    </tr>
    <tr>
      <th>1351</th>
      <td>704054845121142784</td>
      <td>Here is a whole flock of puppers.  60/50 I'll take the lot https://t.co/9dpcw6MdWa</td>
      <td>60</td>
      <td>50</td>
    </tr>
    <tr>
      <th>1433</th>
      <td>697463031882764288</td>
      <td>Happy Wednesday here's a bucket of pups. 44/40 would pet all at once https://t.co/HppvrYuamZ</td>
      <td>44</td>
      <td>40</td>
    </tr>
    <tr>
      <th>1598</th>
      <td>686035780142297088</td>
      <td>Yes I do realize a rating of 4/20 would've been fitting. However, it would be unjust to give these cooperative pups that low of a rating</td>
      <td>4</td>
      <td>20</td>
    </tr>
    <tr>
      <th>1634</th>
      <td>684225744407494656</td>
      <td>Two sneaky puppers were not initially seen, moving the rating to 143/130. Please forgive us. Thank you https://t.co/kRK51Y5ac3</td>
      <td>143</td>
      <td>130</td>
    </tr>
    <tr>
      <th>1635</th>
      <td>684222868335505415</td>
      <td>Someone help the girl is being mugged. Several are distracting her while two steal her shoes. Clever puppers 121/110 https://t.co/1zfnTJLt55</td>
      <td>121</td>
      <td>110</td>
    </tr>
    <tr>
      <th>1662</th>
      <td>682962037429899265</td>
      <td>This is Darrel. He just robbed a 7/11 and is in a high speed police chase. Was just spotted by the helicopter 10/10 https://t.co/7EsP8LmSp5</td>
      <td>7</td>
      <td>11</td>
    </tr>
    <tr>
      <th>1663</th>
      <td>682808988178739200</td>
      <td>I'm aware that I could've said 20/16, but here at WeRateDogs we are very professional. An inconsistent rating scale is simply irresponsible</td>
      <td>20</td>
      <td>16</td>
    </tr>
    <tr>
      <th>1779</th>
      <td>677716515794329600</td>
      <td>IT'S PUPPERGEDDON. Total of 144/120 ...I think https://t.co/ZanVtAtvIq</td>
      <td>144</td>
      <td>120</td>
    </tr>
    <tr>
      <th>1843</th>
      <td>675853064436391936</td>
      <td>Here we have an entire platoon of puppers. Total score: 88/80 would pet all at once https://t.co/y93p6FLvVw</td>
      <td>88</td>
      <td>80</td>
    </tr>
    <tr>
      <th>2335</th>
      <td>666287406224695296</td>
      <td>This is an Albanian 3 1/2 legged  Episcopalian. Loves well-polished hardwood flooring. Penis on the collar. 9/10 https://t.co/d9NcXFKwLv</td>
      <td>1</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Check for duplicates
twitter_archive[twitter_archive.tweet_id.duplicated()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
# Count the number of NaNs in each column
twitter_archive.isnull().sum()
```




    tweet_id                      0   
    in_reply_to_status_id         2278
    in_reply_to_user_id           2278
    timestamp                     0   
    source                        0   
    text                          0   
    retweeted_status_id           2175
    retweeted_status_user_id      2175
    retweeted_status_timestamp    2175
    expanded_urls                 59  
    rating_numerator              0   
    rating_denominator            0   
    name                          0   
    doggo                         0   
    floofer                       0   
    pupper                        0   
    puppo                         0   
    dtype: int64




```python
# There were some tweet ids which we could not scrape using the API
errors_df = pd.read_csv('tweet_json_errors.csv')
errors_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>888202515573088257</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-21 01:02:36 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: This is Canela. She attempted some fancy porch pics. They were unsuccessful. 13/10 someone help her https://t.co/cLyzpcUcMX</td>
      <td>8.874740e+17</td>
      <td>4.196984e+09</td>
      <td>2017-07-19 00:47:34 +0000</td>
      <td>https://twitter.com/dog_rates/status/887473957103951883/photo/1,https://twitter.com/dog_rates/status/887473957103951883/photo/1,https://twitter.com/dog_rates/status/887473957103951883/photo/1,https://twitter.com/dog_rates/status/887473957103951883/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Canela</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>873697596434513921</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-11 00:25:14 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: This is Walter. He won't start hydrotherapy without his favorite floatie. 14/10 keep it pup Walter https://t.co/r28jFx9uyF</td>
      <td>8.688804e+17</td>
      <td>4.196984e+09</td>
      <td>2017-05-28 17:23:24 +0000</td>
      <td>https://twitter.com/dog_rates/status/868880397819494401/photo/1,https://twitter.com/dog_rates/status/868880397819494401/photo/1</td>
      <td>14</td>
      <td>10</td>
      <td>Walter</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>869988702071779329</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-05-31 18:47:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: We only rate dogs. This is quite clearly a smol broken polar bear. We'd appreciate if you only send dogs. Thank you... 12/10…</td>
      <td>8.591970e+17</td>
      <td>4.196984e+09</td>
      <td>2017-05-02 00:04:57 +0000</td>
      <td>https://twitter.com/dog_rates/status/859196978902773760/video/1</td>
      <td>12</td>
      <td>10</td>
      <td>quite</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>866816280283807744</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-05-23 00:41:20 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: This is Jamesy. He gives a kiss to every other pupper he sees on his walk. 13/10 such passion, much tender https://t.co/wk7T…</td>
      <td>8.664507e+17</td>
      <td>4.196984e+09</td>
      <td>2017-05-22 00:28:40 +0000</td>
      <td>https://twitter.com/dog_rates/status/866450705531457537/photo/1,https://twitter.com/dog_rates/status/866450705531457537/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Jamesy</td>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>861769973181624320</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-05-09 02:29:07 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: "Good afternoon class today we're going to learn what makes a good boy so good" 13/10 https://t.co/f1h2Fsalv9</td>
      <td>8.066291e+17</td>
      <td>4.196984e+09</td>
      <td>2016-12-07 22:38:52 +0000</td>
      <td>https://twitter.com/dog_rates/status/806629075125202948/photo/1,https://twitter.com/dog_rates/status/806629075125202948/photo/1,https://twitter.com/dog_rates/status/806629075125202948/photo/1,https://twitter.com/dog_rates/status/806629075125202948/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>5</th>
      <td>845459076796616705</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-03-25 02:15:26 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: Here's a heartwarming scene of a single father raising his two pups. Downright awe-inspiring af. 12/10 for everyone https://…</td>
      <td>7.562885e+17</td>
      <td>4.196984e+09</td>
      <td>2016-07-22 00:43:32 +0000</td>
      <td>https://twitter.com/dog_rates/status/756288534030475264/photo/1,https://twitter.com/dog_rates/status/756288534030475264/photo/1,https://twitter.com/dog_rates/status/756288534030475264/photo/1,https://twitter.com/dog_rates/status/756288534030475264/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>6</th>
      <td>842892208864923648</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-03-18 00:15:37 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: This is Stephan. He just wants to help. 13/10 such a good boy https://t.co/DkBYaCAg2d</td>
      <td>8.071068e+17</td>
      <td>4.196984e+09</td>
      <td>2016-12-09 06:17:20 +0000</td>
      <td>https://twitter.com/dog_rates/status/807106840509214720/video/1,https://twitter.com/dog_rates/status/807106840509214720/video/1</td>
      <td>13</td>
      <td>10</td>
      <td>Stephan</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>7</th>
      <td>837012587749474308</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-03-01 18:52:06 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @KennyFromDaBlok: 14/10 h*ckin good hats. will wear daily @dog_rates https://t.co/rHLoU5gS30</td>
      <td>8.370113e+17</td>
      <td>7.266347e+08</td>
      <td>2017-03-01 18:47:10 +0000</td>
      <td>https://twitter.com/KennyFromDaBlok/status/837011344666812416/photo/1,https://twitter.com/KennyFromDaBlok/status/837011344666812416/photo/1</td>
      <td>14</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>8</th>
      <td>827228250799742977</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-02-02 18:52:38 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: This is Phil. He's an important dog. Can control the seasons. Magical as hell. 12/10 would let him sign my forehead https://…</td>
      <td>6.946697e+17</td>
      <td>4.196984e+09</td>
      <td>2016-02-02 23:52:22 +0000</td>
      <td>https://twitter.com/dog_rates/status/694669722378485760/photo/1,https://twitter.com/dog_rates/status/694669722378485760/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Phil</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>9</th>
      <td>802247111496568832</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-11-25 20:26:31 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: Everybody drop what you're doing and look at this dog. 13/10 must be super h*ckin rare https://t.co/I1bJUzUEW5</td>
      <td>7.790561e+17</td>
      <td>4.196984e+09</td>
      <td>2016-09-22 20:33:42 +0000</td>
      <td>https://twitter.com/dog_rates/status/779056095788752897/photo/1,https://twitter.com/dog_rates/status/779056095788752897/photo/1,https://twitter.com/dog_rates/status/779056095788752897/photo/1,https://twitter.com/dog_rates/status/779056095788752897/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>10</th>
      <td>775096608509886464</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-09-11 22:20:06 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: After so many requests, this is Bretagne. She was the last surviving 9/11 search dog, and our second ever 14/10. RIP https:/…</td>
      <td>7.403732e+17</td>
      <td>4.196984e+09</td>
      <td>2016-06-08 02:41:38 +0000</td>
      <td>https://twitter.com/dog_rates/status/740373189193256964/photo/1,https://twitter.com/dog_rates/status/740373189193256964/photo/1,https://twitter.com/dog_rates/status/740373189193256964/photo/1,https://twitter.com/dog_rates/status/740373189193256964/photo/1</td>
      <td>9</td>
      <td>11</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
image_predictions
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>jpg_url</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>666020888022790149</td>
      <td>https://pbs.twimg.com/media/CT4udn0WwAA0aMy.jpg</td>
      <td>1</td>
      <td>Welsh_springer_spaniel</td>
      <td>0.465074</td>
      <td>True</td>
      <td>collie</td>
      <td>0.156665</td>
      <td>True</td>
      <td>Shetland_sheepdog</td>
      <td>0.061428</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>666029285002620928</td>
      <td>https://pbs.twimg.com/media/CT42GRgUYAA5iDo.jpg</td>
      <td>1</td>
      <td>redbone</td>
      <td>0.506826</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.074192</td>
      <td>True</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.072010</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>666033412701032449</td>
      <td>https://pbs.twimg.com/media/CT4521TWwAEvMyu.jpg</td>
      <td>1</td>
      <td>German_shepherd</td>
      <td>0.596461</td>
      <td>True</td>
      <td>malinois</td>
      <td>0.138584</td>
      <td>True</td>
      <td>bloodhound</td>
      <td>0.116197</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>666044226329800704</td>
      <td>https://pbs.twimg.com/media/CT5Dr8HUEAA-lEu.jpg</td>
      <td>1</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.408143</td>
      <td>True</td>
      <td>redbone</td>
      <td>0.360687</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.222752</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>666049248165822465</td>
      <td>https://pbs.twimg.com/media/CT5IQmsXIAAKY4A.jpg</td>
      <td>1</td>
      <td>miniature_pinscher</td>
      <td>0.560311</td>
      <td>True</td>
      <td>Rottweiler</td>
      <td>0.243682</td>
      <td>True</td>
      <td>Doberman</td>
      <td>0.154629</td>
      <td>True</td>
    </tr>
    <tr>
      <th>5</th>
      <td>666050758794694657</td>
      <td>https://pbs.twimg.com/media/CT5Jof1WUAEuVxN.jpg</td>
      <td>1</td>
      <td>Bernese_mountain_dog</td>
      <td>0.651137</td>
      <td>True</td>
      <td>English_springer</td>
      <td>0.263788</td>
      <td>True</td>
      <td>Greater_Swiss_Mountain_dog</td>
      <td>0.016199</td>
      <td>True</td>
    </tr>
    <tr>
      <th>6</th>
      <td>666051853826850816</td>
      <td>https://pbs.twimg.com/media/CT5KoJ1WoAAJash.jpg</td>
      <td>1</td>
      <td>box_turtle</td>
      <td>0.933012</td>
      <td>False</td>
      <td>mud_turtle</td>
      <td>0.045885</td>
      <td>False</td>
      <td>terrapin</td>
      <td>0.017885</td>
      <td>False</td>
    </tr>
    <tr>
      <th>7</th>
      <td>666055525042405380</td>
      <td>https://pbs.twimg.com/media/CT5N9tpXIAAifs1.jpg</td>
      <td>1</td>
      <td>chow</td>
      <td>0.692517</td>
      <td>True</td>
      <td>Tibetan_mastiff</td>
      <td>0.058279</td>
      <td>True</td>
      <td>fur_coat</td>
      <td>0.054449</td>
      <td>False</td>
    </tr>
    <tr>
      <th>8</th>
      <td>666057090499244032</td>
      <td>https://pbs.twimg.com/media/CT5PY90WoAAQGLo.jpg</td>
      <td>1</td>
      <td>shopping_cart</td>
      <td>0.962465</td>
      <td>False</td>
      <td>shopping_basket</td>
      <td>0.014594</td>
      <td>False</td>
      <td>golden_retriever</td>
      <td>0.007959</td>
      <td>True</td>
    </tr>
    <tr>
      <th>9</th>
      <td>666058600524156928</td>
      <td>https://pbs.twimg.com/media/CT5Qw94XAAA_2dP.jpg</td>
      <td>1</td>
      <td>miniature_poodle</td>
      <td>0.201493</td>
      <td>True</td>
      <td>komondor</td>
      <td>0.192305</td>
      <td>True</td>
      <td>soft-coated_wheaten_terrier</td>
      <td>0.082086</td>
      <td>True</td>
    </tr>
    <tr>
      <th>10</th>
      <td>666063827256086533</td>
      <td>https://pbs.twimg.com/media/CT5Vg_wXIAAXfnj.jpg</td>
      <td>1</td>
      <td>golden_retriever</td>
      <td>0.775930</td>
      <td>True</td>
      <td>Tibetan_mastiff</td>
      <td>0.093718</td>
      <td>True</td>
      <td>Labrador_retriever</td>
      <td>0.072427</td>
      <td>True</td>
    </tr>
    <tr>
      <th>11</th>
      <td>666071193221509120</td>
      <td>https://pbs.twimg.com/media/CT5cN_3WEAAlOoZ.jpg</td>
      <td>1</td>
      <td>Gordon_setter</td>
      <td>0.503672</td>
      <td>True</td>
      <td>Yorkshire_terrier</td>
      <td>0.174201</td>
      <td>True</td>
      <td>Pekinese</td>
      <td>0.109454</td>
      <td>True</td>
    </tr>
    <tr>
      <th>12</th>
      <td>666073100786774016</td>
      <td>https://pbs.twimg.com/media/CT5d9DZXAAALcwe.jpg</td>
      <td>1</td>
      <td>Walker_hound</td>
      <td>0.260857</td>
      <td>True</td>
      <td>English_foxhound</td>
      <td>0.175382</td>
      <td>True</td>
      <td>Ibizan_hound</td>
      <td>0.097471</td>
      <td>True</td>
    </tr>
    <tr>
      <th>13</th>
      <td>666082916733198337</td>
      <td>https://pbs.twimg.com/media/CT5m4VGWEAAtKc8.jpg</td>
      <td>1</td>
      <td>pug</td>
      <td>0.489814</td>
      <td>True</td>
      <td>bull_mastiff</td>
      <td>0.404722</td>
      <td>True</td>
      <td>French_bulldog</td>
      <td>0.048960</td>
      <td>True</td>
    </tr>
    <tr>
      <th>14</th>
      <td>666094000022159362</td>
      <td>https://pbs.twimg.com/media/CT5w9gUW4AAsBNN.jpg</td>
      <td>1</td>
      <td>bloodhound</td>
      <td>0.195217</td>
      <td>True</td>
      <td>German_shepherd</td>
      <td>0.078260</td>
      <td>True</td>
      <td>malinois</td>
      <td>0.075628</td>
      <td>True</td>
    </tr>
    <tr>
      <th>15</th>
      <td>666099513787052032</td>
      <td>https://pbs.twimg.com/media/CT51-JJUEAA6hV8.jpg</td>
      <td>1</td>
      <td>Lhasa</td>
      <td>0.582330</td>
      <td>True</td>
      <td>Shih-Tzu</td>
      <td>0.166192</td>
      <td>True</td>
      <td>Dandie_Dinmont</td>
      <td>0.089688</td>
      <td>True</td>
    </tr>
    <tr>
      <th>16</th>
      <td>666102155909144576</td>
      <td>https://pbs.twimg.com/media/CT54YGiWUAEZnoK.jpg</td>
      <td>1</td>
      <td>English_setter</td>
      <td>0.298617</td>
      <td>True</td>
      <td>Newfoundland</td>
      <td>0.149842</td>
      <td>True</td>
      <td>borzoi</td>
      <td>0.133649</td>
      <td>True</td>
    </tr>
    <tr>
      <th>17</th>
      <td>666104133288665088</td>
      <td>https://pbs.twimg.com/media/CT56LSZWoAAlJj2.jpg</td>
      <td>1</td>
      <td>hen</td>
      <td>0.965932</td>
      <td>False</td>
      <td>cock</td>
      <td>0.033919</td>
      <td>False</td>
      <td>partridge</td>
      <td>0.000052</td>
      <td>False</td>
    </tr>
    <tr>
      <th>18</th>
      <td>666268910803644416</td>
      <td>https://pbs.twimg.com/media/CT8QCd1WEAADXws.jpg</td>
      <td>1</td>
      <td>desktop_computer</td>
      <td>0.086502</td>
      <td>False</td>
      <td>desk</td>
      <td>0.085547</td>
      <td>False</td>
      <td>bookcase</td>
      <td>0.079480</td>
      <td>False</td>
    </tr>
    <tr>
      <th>19</th>
      <td>666273097616637952</td>
      <td>https://pbs.twimg.com/media/CT8T1mtUwAA3aqm.jpg</td>
      <td>1</td>
      <td>Italian_greyhound</td>
      <td>0.176053</td>
      <td>True</td>
      <td>toy_terrier</td>
      <td>0.111884</td>
      <td>True</td>
      <td>basenji</td>
      <td>0.111152</td>
      <td>True</td>
    </tr>
    <tr>
      <th>20</th>
      <td>666287406224695296</td>
      <td>https://pbs.twimg.com/media/CT8g3BpUEAAuFjg.jpg</td>
      <td>1</td>
      <td>Maltese_dog</td>
      <td>0.857531</td>
      <td>True</td>
      <td>toy_poodle</td>
      <td>0.063064</td>
      <td>True</td>
      <td>miniature_poodle</td>
      <td>0.025581</td>
      <td>True</td>
    </tr>
    <tr>
      <th>21</th>
      <td>666293911632134144</td>
      <td>https://pbs.twimg.com/media/CT8mx7KW4AEQu8N.jpg</td>
      <td>1</td>
      <td>three-toed_sloth</td>
      <td>0.914671</td>
      <td>False</td>
      <td>otter</td>
      <td>0.015250</td>
      <td>False</td>
      <td>great_grey_owl</td>
      <td>0.013207</td>
      <td>False</td>
    </tr>
    <tr>
      <th>22</th>
      <td>666337882303524864</td>
      <td>https://pbs.twimg.com/media/CT9OwFIWEAMuRje.jpg</td>
      <td>1</td>
      <td>ox</td>
      <td>0.416669</td>
      <td>False</td>
      <td>Newfoundland</td>
      <td>0.278407</td>
      <td>True</td>
      <td>groenendael</td>
      <td>0.102643</td>
      <td>True</td>
    </tr>
    <tr>
      <th>23</th>
      <td>666345417576210432</td>
      <td>https://pbs.twimg.com/media/CT9Vn7PWoAA_ZCM.jpg</td>
      <td>1</td>
      <td>golden_retriever</td>
      <td>0.858744</td>
      <td>True</td>
      <td>Chesapeake_Bay_retriever</td>
      <td>0.054787</td>
      <td>True</td>
      <td>Labrador_retriever</td>
      <td>0.014241</td>
      <td>True</td>
    </tr>
    <tr>
      <th>24</th>
      <td>666353288456101888</td>
      <td>https://pbs.twimg.com/media/CT9cx0tUEAAhNN_.jpg</td>
      <td>1</td>
      <td>malamute</td>
      <td>0.336874</td>
      <td>True</td>
      <td>Siberian_husky</td>
      <td>0.147655</td>
      <td>True</td>
      <td>Eskimo_dog</td>
      <td>0.093412</td>
      <td>True</td>
    </tr>
    <tr>
      <th>25</th>
      <td>666362758909284353</td>
      <td>https://pbs.twimg.com/media/CT9lXGsUcAAyUFt.jpg</td>
      <td>1</td>
      <td>guinea_pig</td>
      <td>0.996496</td>
      <td>False</td>
      <td>skunk</td>
      <td>0.002402</td>
      <td>False</td>
      <td>hamster</td>
      <td>0.000461</td>
      <td>False</td>
    </tr>
    <tr>
      <th>26</th>
      <td>666373753744588802</td>
      <td>https://pbs.twimg.com/media/CT9vZEYWUAAlZ05.jpg</td>
      <td>1</td>
      <td>soft-coated_wheaten_terrier</td>
      <td>0.326467</td>
      <td>True</td>
      <td>Afghan_hound</td>
      <td>0.259551</td>
      <td>True</td>
      <td>briard</td>
      <td>0.206803</td>
      <td>True</td>
    </tr>
    <tr>
      <th>27</th>
      <td>666396247373291520</td>
      <td>https://pbs.twimg.com/media/CT-D2ZHWIAA3gK1.jpg</td>
      <td>1</td>
      <td>Chihuahua</td>
      <td>0.978108</td>
      <td>True</td>
      <td>toy_terrier</td>
      <td>0.009397</td>
      <td>True</td>
      <td>papillon</td>
      <td>0.004577</td>
      <td>True</td>
    </tr>
    <tr>
      <th>28</th>
      <td>666407126856765440</td>
      <td>https://pbs.twimg.com/media/CT-NvwmW4AAugGZ.jpg</td>
      <td>1</td>
      <td>black-and-tan_coonhound</td>
      <td>0.529139</td>
      <td>True</td>
      <td>bloodhound</td>
      <td>0.244220</td>
      <td>True</td>
      <td>flat-coated_retriever</td>
      <td>0.173810</td>
      <td>True</td>
    </tr>
    <tr>
      <th>29</th>
      <td>666411507551481857</td>
      <td>https://pbs.twimg.com/media/CT-RugiWIAELEaq.jpg</td>
      <td>1</td>
      <td>coho</td>
      <td>0.404640</td>
      <td>False</td>
      <td>barracouta</td>
      <td>0.271485</td>
      <td>False</td>
      <td>gar</td>
      <td>0.189945</td>
      <td>False</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2045</th>
      <td>886366144734445568</td>
      <td>https://pbs.twimg.com/media/DE0BTnQUwAApKEH.jpg</td>
      <td>1</td>
      <td>French_bulldog</td>
      <td>0.999201</td>
      <td>True</td>
      <td>Chihuahua</td>
      <td>0.000361</td>
      <td>True</td>
      <td>Boston_bull</td>
      <td>0.000076</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2046</th>
      <td>886680336477933568</td>
      <td>https://pbs.twimg.com/media/DE4fEDzWAAAyHMM.jpg</td>
      <td>1</td>
      <td>convertible</td>
      <td>0.738995</td>
      <td>False</td>
      <td>sports_car</td>
      <td>0.139952</td>
      <td>False</td>
      <td>car_wheel</td>
      <td>0.044173</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2047</th>
      <td>886736880519319552</td>
      <td>https://pbs.twimg.com/media/DE5Se8FXcAAJFx4.jpg</td>
      <td>1</td>
      <td>kuvasz</td>
      <td>0.309706</td>
      <td>True</td>
      <td>Great_Pyrenees</td>
      <td>0.186136</td>
      <td>True</td>
      <td>Dandie_Dinmont</td>
      <td>0.086346</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2048</th>
      <td>886983233522544640</td>
      <td>https://pbs.twimg.com/media/DE8yicJW0AAAvBJ.jpg</td>
      <td>2</td>
      <td>Chihuahua</td>
      <td>0.793469</td>
      <td>True</td>
      <td>toy_terrier</td>
      <td>0.143528</td>
      <td>True</td>
      <td>can_opener</td>
      <td>0.032253</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2049</th>
      <td>887101392804085760</td>
      <td>https://pbs.twimg.com/media/DE-eAq6UwAA-jaE.jpg</td>
      <td>1</td>
      <td>Samoyed</td>
      <td>0.733942</td>
      <td>True</td>
      <td>Eskimo_dog</td>
      <td>0.035029</td>
      <td>True</td>
      <td>Staffordshire_bullterrier</td>
      <td>0.029705</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2050</th>
      <td>887343217045368832</td>
      <td>https://pbs.twimg.com/ext_tw_video_thumb/887343120832229379/pu/img/6HSuFrW1lzI_9Mht.jpg</td>
      <td>1</td>
      <td>Mexican_hairless</td>
      <td>0.330741</td>
      <td>True</td>
      <td>sea_lion</td>
      <td>0.275645</td>
      <td>False</td>
      <td>Weimaraner</td>
      <td>0.134203</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2051</th>
      <td>887473957103951883</td>
      <td>https://pbs.twimg.com/media/DFDw2tyUQAAAFke.jpg</td>
      <td>2</td>
      <td>Pembroke</td>
      <td>0.809197</td>
      <td>True</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.054950</td>
      <td>True</td>
      <td>beagle</td>
      <td>0.038915</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2052</th>
      <td>887517139158093824</td>
      <td>https://pbs.twimg.com/ext_tw_video_thumb/887517108413886465/pu/img/WanJKwssZj4VJvL9.jpg</td>
      <td>1</td>
      <td>limousine</td>
      <td>0.130432</td>
      <td>False</td>
      <td>tow_truck</td>
      <td>0.029175</td>
      <td>False</td>
      <td>shopping_cart</td>
      <td>0.026321</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2053</th>
      <td>887705289381826560</td>
      <td>https://pbs.twimg.com/media/DFHDQBbXgAEqY7t.jpg</td>
      <td>1</td>
      <td>basset</td>
      <td>0.821664</td>
      <td>True</td>
      <td>redbone</td>
      <td>0.087582</td>
      <td>True</td>
      <td>Weimaraner</td>
      <td>0.026236</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2054</th>
      <td>888078434458587136</td>
      <td>https://pbs.twimg.com/media/DFMWn56WsAAkA7B.jpg</td>
      <td>1</td>
      <td>French_bulldog</td>
      <td>0.995026</td>
      <td>True</td>
      <td>pug</td>
      <td>0.000932</td>
      <td>True</td>
      <td>bull_mastiff</td>
      <td>0.000903</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2055</th>
      <td>888202515573088257</td>
      <td>https://pbs.twimg.com/media/DFDw2tyUQAAAFke.jpg</td>
      <td>2</td>
      <td>Pembroke</td>
      <td>0.809197</td>
      <td>True</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.054950</td>
      <td>True</td>
      <td>beagle</td>
      <td>0.038915</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2056</th>
      <td>888554962724278272</td>
      <td>https://pbs.twimg.com/media/DFTH_O-UQAACu20.jpg</td>
      <td>3</td>
      <td>Siberian_husky</td>
      <td>0.700377</td>
      <td>True</td>
      <td>Eskimo_dog</td>
      <td>0.166511</td>
      <td>True</td>
      <td>malamute</td>
      <td>0.111411</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2057</th>
      <td>888804989199671297</td>
      <td>https://pbs.twimg.com/media/DFWra-3VYAA2piG.jpg</td>
      <td>1</td>
      <td>golden_retriever</td>
      <td>0.469760</td>
      <td>True</td>
      <td>Labrador_retriever</td>
      <td>0.184172</td>
      <td>True</td>
      <td>English_setter</td>
      <td>0.073482</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2058</th>
      <td>888917238123831296</td>
      <td>https://pbs.twimg.com/media/DFYRgsOUQAARGhO.jpg</td>
      <td>1</td>
      <td>golden_retriever</td>
      <td>0.714719</td>
      <td>True</td>
      <td>Tibetan_mastiff</td>
      <td>0.120184</td>
      <td>True</td>
      <td>Labrador_retriever</td>
      <td>0.105506</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2059</th>
      <td>889278841981685760</td>
      <td>https://pbs.twimg.com/ext_tw_video_thumb/889278779352338437/pu/img/VlbFB3v8H8VwzVNY.jpg</td>
      <td>1</td>
      <td>whippet</td>
      <td>0.626152</td>
      <td>True</td>
      <td>borzoi</td>
      <td>0.194742</td>
      <td>True</td>
      <td>Saluki</td>
      <td>0.027351</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2060</th>
      <td>889531135344209921</td>
      <td>https://pbs.twimg.com/media/DFg_2PVW0AEHN3p.jpg</td>
      <td>1</td>
      <td>golden_retriever</td>
      <td>0.953442</td>
      <td>True</td>
      <td>Labrador_retriever</td>
      <td>0.013834</td>
      <td>True</td>
      <td>redbone</td>
      <td>0.007958</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2061</th>
      <td>889638837579907072</td>
      <td>https://pbs.twimg.com/media/DFihzFfXsAYGDPR.jpg</td>
      <td>1</td>
      <td>French_bulldog</td>
      <td>0.991650</td>
      <td>True</td>
      <td>boxer</td>
      <td>0.002129</td>
      <td>True</td>
      <td>Staffordshire_bullterrier</td>
      <td>0.001498</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2062</th>
      <td>889665388333682689</td>
      <td>https://pbs.twimg.com/media/DFi579UWsAAatzw.jpg</td>
      <td>1</td>
      <td>Pembroke</td>
      <td>0.966327</td>
      <td>True</td>
      <td>Cardigan</td>
      <td>0.027356</td>
      <td>True</td>
      <td>basenji</td>
      <td>0.004633</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2063</th>
      <td>889880896479866881</td>
      <td>https://pbs.twimg.com/media/DFl99B1WsAITKsg.jpg</td>
      <td>1</td>
      <td>French_bulldog</td>
      <td>0.377417</td>
      <td>True</td>
      <td>Labrador_retriever</td>
      <td>0.151317</td>
      <td>True</td>
      <td>muzzle</td>
      <td>0.082981</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2064</th>
      <td>890006608113172480</td>
      <td>https://pbs.twimg.com/media/DFnwSY4WAAAMliS.jpg</td>
      <td>1</td>
      <td>Samoyed</td>
      <td>0.957979</td>
      <td>True</td>
      <td>Pomeranian</td>
      <td>0.013884</td>
      <td>True</td>
      <td>chow</td>
      <td>0.008167</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2065</th>
      <td>890240255349198849</td>
      <td>https://pbs.twimg.com/media/DFrEyVuW0AAO3t9.jpg</td>
      <td>1</td>
      <td>Pembroke</td>
      <td>0.511319</td>
      <td>True</td>
      <td>Cardigan</td>
      <td>0.451038</td>
      <td>True</td>
      <td>Chihuahua</td>
      <td>0.029248</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2066</th>
      <td>890609185150312448</td>
      <td>https://pbs.twimg.com/media/DFwUU__XcAEpyXI.jpg</td>
      <td>1</td>
      <td>Irish_terrier</td>
      <td>0.487574</td>
      <td>True</td>
      <td>Irish_setter</td>
      <td>0.193054</td>
      <td>True</td>
      <td>Chesapeake_Bay_retriever</td>
      <td>0.118184</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2067</th>
      <td>890729181411237888</td>
      <td>https://pbs.twimg.com/media/DFyBahAVwAAhUTd.jpg</td>
      <td>2</td>
      <td>Pomeranian</td>
      <td>0.566142</td>
      <td>True</td>
      <td>Eskimo_dog</td>
      <td>0.178406</td>
      <td>True</td>
      <td>Pembroke</td>
      <td>0.076507</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2068</th>
      <td>890971913173991426</td>
      <td>https://pbs.twimg.com/media/DF1eOmZXUAALUcq.jpg</td>
      <td>1</td>
      <td>Appenzeller</td>
      <td>0.341703</td>
      <td>True</td>
      <td>Border_collie</td>
      <td>0.199287</td>
      <td>True</td>
      <td>ice_lolly</td>
      <td>0.193548</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2069</th>
      <td>891087950875897856</td>
      <td>https://pbs.twimg.com/media/DF3HwyEWsAABqE6.jpg</td>
      <td>1</td>
      <td>Chesapeake_Bay_retriever</td>
      <td>0.425595</td>
      <td>True</td>
      <td>Irish_terrier</td>
      <td>0.116317</td>
      <td>True</td>
      <td>Indian_elephant</td>
      <td>0.076902</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2070</th>
      <td>891327558926688256</td>
      <td>https://pbs.twimg.com/media/DF6hr6BUMAAzZgT.jpg</td>
      <td>2</td>
      <td>basset</td>
      <td>0.555712</td>
      <td>True</td>
      <td>English_springer</td>
      <td>0.225770</td>
      <td>True</td>
      <td>German_short-haired_pointer</td>
      <td>0.175219</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2071</th>
      <td>891689557279858688</td>
      <td>https://pbs.twimg.com/media/DF_q7IAWsAEuuN8.jpg</td>
      <td>1</td>
      <td>paper_towel</td>
      <td>0.170278</td>
      <td>False</td>
      <td>Labrador_retriever</td>
      <td>0.168086</td>
      <td>True</td>
      <td>spatula</td>
      <td>0.040836</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2072</th>
      <td>891815181378084864</td>
      <td>https://pbs.twimg.com/media/DGBdLU1WsAANxJ9.jpg</td>
      <td>1</td>
      <td>Chihuahua</td>
      <td>0.716012</td>
      <td>True</td>
      <td>malamute</td>
      <td>0.078253</td>
      <td>True</td>
      <td>kelpie</td>
      <td>0.031379</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2073</th>
      <td>892177421306343426</td>
      <td>https://pbs.twimg.com/media/DGGmoV4XsAAUL6n.jpg</td>
      <td>1</td>
      <td>Chihuahua</td>
      <td>0.323581</td>
      <td>True</td>
      <td>Pekinese</td>
      <td>0.090647</td>
      <td>True</td>
      <td>papillon</td>
      <td>0.068957</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2074</th>
      <td>892420643555336193</td>
      <td>https://pbs.twimg.com/media/DGKD1-bXoAAIAUK.jpg</td>
      <td>1</td>
      <td>orange</td>
      <td>0.097049</td>
      <td>False</td>
      <td>bagel</td>
      <td>0.085851</td>
      <td>False</td>
      <td>banana</td>
      <td>0.076110</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>2075 rows × 12 columns</p>
</div>




```python
# Check the predicyion confidence values
image_predictions.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>img_num</th>
      <th>p1_conf</th>
      <th>p2_conf</th>
      <th>p3_conf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.075000e+03</td>
      <td>2075.000000</td>
      <td>2075.000000</td>
      <td>2.075000e+03</td>
      <td>2.075000e+03</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>7.384514e+17</td>
      <td>1.203855</td>
      <td>0.594548</td>
      <td>1.345886e-01</td>
      <td>6.032417e-02</td>
    </tr>
    <tr>
      <th>std</th>
      <td>6.785203e+16</td>
      <td>0.561875</td>
      <td>0.271174</td>
      <td>1.006657e-01</td>
      <td>5.090593e-02</td>
    </tr>
    <tr>
      <th>min</th>
      <td>6.660209e+17</td>
      <td>1.000000</td>
      <td>0.044333</td>
      <td>1.011300e-08</td>
      <td>1.740170e-10</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>6.764835e+17</td>
      <td>1.000000</td>
      <td>0.364412</td>
      <td>5.388625e-02</td>
      <td>1.622240e-02</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7.119988e+17</td>
      <td>1.000000</td>
      <td>0.588230</td>
      <td>1.181810e-01</td>
      <td>4.944380e-02</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.932034e+17</td>
      <td>1.000000</td>
      <td>0.843855</td>
      <td>1.955655e-01</td>
      <td>9.180755e-02</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8.924206e+17</td>
      <td>4.000000</td>
      <td>1.000000</td>
      <td>4.880140e-01</td>
      <td>2.734190e-01</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Looking for duplicated Tweet IDs
image_predictions[image_predictions.tweet_id.duplicated()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>jpg_url</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
tweet_json
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>favorite_count</th>
      <th>retweet_count</th>
      <th>tweet_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>38824</td>
      <td>8591</td>
      <td>892420643555336193</td>
    </tr>
    <tr>
      <th>1</th>
      <td>33253</td>
      <td>6312</td>
      <td>892177421306343426</td>
    </tr>
    <tr>
      <th>2</th>
      <td>25044</td>
      <td>4190</td>
      <td>891815181378084864</td>
    </tr>
    <tr>
      <th>3</th>
      <td>42188</td>
      <td>8706</td>
      <td>891689557279858688</td>
    </tr>
    <tr>
      <th>4</th>
      <td>40347</td>
      <td>9474</td>
      <td>891327558926688256</td>
    </tr>
    <tr>
      <th>5</th>
      <td>20236</td>
      <td>3137</td>
      <td>891087950875897856</td>
    </tr>
    <tr>
      <th>6</th>
      <td>11861</td>
      <td>2088</td>
      <td>890971913173991426</td>
    </tr>
    <tr>
      <th>7</th>
      <td>65572</td>
      <td>19045</td>
      <td>890729181411237888</td>
    </tr>
    <tr>
      <th>8</th>
      <td>27786</td>
      <td>4300</td>
      <td>890609185150312448</td>
    </tr>
    <tr>
      <th>9</th>
      <td>31956</td>
      <td>7474</td>
      <td>890240255349198849</td>
    </tr>
    <tr>
      <th>10</th>
      <td>30670</td>
      <td>7394</td>
      <td>890006608113172480</td>
    </tr>
    <tr>
      <th>11</th>
      <td>27807</td>
      <td>5008</td>
      <td>889880896479866881</td>
    </tr>
    <tr>
      <th>12</th>
      <td>48159</td>
      <td>10141</td>
      <td>889665388333682689</td>
    </tr>
    <tr>
      <th>13</th>
      <td>27193</td>
      <td>4578</td>
      <td>889638837579907072</td>
    </tr>
    <tr>
      <th>14</th>
      <td>15108</td>
      <td>2252</td>
      <td>889531135344209921</td>
    </tr>
    <tr>
      <th>15</th>
      <td>25322</td>
      <td>5475</td>
      <td>889278841981685760</td>
    </tr>
    <tr>
      <th>16</th>
      <td>29103</td>
      <td>4531</td>
      <td>888917238123831296</td>
    </tr>
    <tr>
      <th>17</th>
      <td>25606</td>
      <td>4379</td>
      <td>888804989199671297</td>
    </tr>
    <tr>
      <th>18</th>
      <td>19926</td>
      <td>3612</td>
      <td>888554962724278272</td>
    </tr>
    <tr>
      <th>19</th>
      <td>21774</td>
      <td>3522</td>
      <td>888078434458587136</td>
    </tr>
    <tr>
      <th>20</th>
      <td>30221</td>
      <td>5438</td>
      <td>887705289381826560</td>
    </tr>
    <tr>
      <th>21</th>
      <td>46262</td>
      <td>11753</td>
      <td>887517139158093824</td>
    </tr>
    <tr>
      <th>22</th>
      <td>69175</td>
      <td>18381</td>
      <td>887473957103951883</td>
    </tr>
    <tr>
      <th>23</th>
      <td>33715</td>
      <td>10490</td>
      <td>887343217045368832</td>
    </tr>
    <tr>
      <th>24</th>
      <td>30566</td>
      <td>6005</td>
      <td>887101392804085760</td>
    </tr>
    <tr>
      <th>25</th>
      <td>35181</td>
      <td>7831</td>
      <td>886983233522544640</td>
    </tr>
    <tr>
      <th>26</th>
      <td>12087</td>
      <td>3322</td>
      <td>886736880519319552</td>
    </tr>
    <tr>
      <th>27</th>
      <td>22447</td>
      <td>4504</td>
      <td>886680336477933568</td>
    </tr>
    <tr>
      <th>28</th>
      <td>21216</td>
      <td>3220</td>
      <td>886366144734445568</td>
    </tr>
    <tr>
      <th>29</th>
      <td>116</td>
      <td>4</td>
      <td>886267009285017600</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2315</th>
      <td>451</td>
      <td>330</td>
      <td>666411507551481857</td>
    </tr>
    <tr>
      <th>2316</th>
      <td>110</td>
      <td>41</td>
      <td>666407126856765440</td>
    </tr>
    <tr>
      <th>2317</th>
      <td>168</td>
      <td>86</td>
      <td>666396247373291520</td>
    </tr>
    <tr>
      <th>2318</th>
      <td>190</td>
      <td>93</td>
      <td>666373753744588802</td>
    </tr>
    <tr>
      <th>2319</th>
      <td>781</td>
      <td>577</td>
      <td>666362758909284353</td>
    </tr>
    <tr>
      <th>2320</th>
      <td>222</td>
      <td>73</td>
      <td>666353288456101888</td>
    </tr>
    <tr>
      <th>2321</th>
      <td>300</td>
      <td>140</td>
      <td>666345417576210432</td>
    </tr>
    <tr>
      <th>2322</th>
      <td>199</td>
      <td>92</td>
      <td>666337882303524864</td>
    </tr>
    <tr>
      <th>2323</th>
      <td>510</td>
      <td>358</td>
      <td>666293911632134144</td>
    </tr>
    <tr>
      <th>2324</th>
      <td>149</td>
      <td>68</td>
      <td>666287406224695296</td>
    </tr>
    <tr>
      <th>2325</th>
      <td>176</td>
      <td>76</td>
      <td>666273097616637952</td>
    </tr>
    <tr>
      <th>2326</th>
      <td>104</td>
      <td>35</td>
      <td>666268910803644416</td>
    </tr>
    <tr>
      <th>2327</th>
      <td>14401</td>
      <td>6660</td>
      <td>666104133288665088</td>
    </tr>
    <tr>
      <th>2328</th>
      <td>80</td>
      <td>13</td>
      <td>666102155909144576</td>
    </tr>
    <tr>
      <th>2329</th>
      <td>158</td>
      <td>69</td>
      <td>666099513787052032</td>
    </tr>
    <tr>
      <th>2330</th>
      <td>165</td>
      <td>76</td>
      <td>666094000022159362</td>
    </tr>
    <tr>
      <th>2331</th>
      <td>119</td>
      <td>45</td>
      <td>666082916733198337</td>
    </tr>
    <tr>
      <th>2332</th>
      <td>323</td>
      <td>166</td>
      <td>666073100786774016</td>
    </tr>
    <tr>
      <th>2333</th>
      <td>148</td>
      <td>62</td>
      <td>666071193221509120</td>
    </tr>
    <tr>
      <th>2334</th>
      <td>479</td>
      <td>223</td>
      <td>666063827256086533</td>
    </tr>
    <tr>
      <th>2335</th>
      <td>112</td>
      <td>57</td>
      <td>666058600524156928</td>
    </tr>
    <tr>
      <th>2336</th>
      <td>298</td>
      <td>142</td>
      <td>666057090499244032</td>
    </tr>
    <tr>
      <th>2337</th>
      <td>437</td>
      <td>257</td>
      <td>666055525042405380</td>
    </tr>
    <tr>
      <th>2338</th>
      <td>1228</td>
      <td>854</td>
      <td>666051853826850816</td>
    </tr>
    <tr>
      <th>2339</th>
      <td>132</td>
      <td>58</td>
      <td>666050758794694657</td>
    </tr>
    <tr>
      <th>2340</th>
      <td>109</td>
      <td>40</td>
      <td>666049248165822465</td>
    </tr>
    <tr>
      <th>2341</th>
      <td>299</td>
      <td>141</td>
      <td>666044226329800704</td>
    </tr>
    <tr>
      <th>2342</th>
      <td>125</td>
      <td>44</td>
      <td>666033412701032449</td>
    </tr>
    <tr>
      <th>2343</th>
      <td>130</td>
      <td>47</td>
      <td>666029285002620928</td>
    </tr>
    <tr>
      <th>2344</th>
      <td>2560</td>
      <td>518</td>
      <td>666020888022790149</td>
    </tr>
  </tbody>
</table>
<p>2345 rows × 3 columns</p>
</div>




```python
tweet_json.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>favorite_count</th>
      <th>retweet_count</th>
      <th>tweet_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2345.000000</td>
      <td>2345.000000</td>
      <td>2.345000e+03</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>8070.775267</td>
      <td>3026.485714</td>
      <td>7.422940e+17</td>
    </tr>
    <tr>
      <th>std</th>
      <td>12143.921062</td>
      <td>5034.152804</td>
      <td>6.833642e+16</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>6.660209e+17</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>1404.000000</td>
      <td>607.000000</td>
      <td>6.783802e+17</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>3539.000000</td>
      <td>1414.000000</td>
      <td>7.189392e+17</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>9979.000000</td>
      <td>3523.000000</td>
      <td>7.986979e+17</td>
    </tr>
    <tr>
      <th>max</th>
      <td>143466.000000</td>
      <td>77385.000000</td>
      <td>8.924206e+17</td>
    </tr>
  </tbody>
</table>
</div>



##### Tidyness Observations
* Relevant data from **twitter_archive**, **image_predictions** and **tweet_json** to be included into a single dataframe.
* From **twitter_archive** create a single categorical variable `dog_stage` and melt `doggo`, `floofer`, `pupper`, `puppo`.
* From **image_predictions** into a new column of final dataframe `dog_breed`, populate it with `p(n)` value, where `p(n)_conf` is highest and `p(n)_dog` is set to True.

##### Quality Observations
* In **twitter_archive** `tweet_id` as well as `in_reply_to_status_id`, `in_reply_to_user_id`, `retweeted_status_id`, `retweeted_status_user_id` all need to be string, as of [Twitter best practices](https://developer.twitter.com/en/docs/tweets/data-dictionary/overview/tweet-object)
* Only original tweets to be used, exclude retweets and replies.
* In **twitter_archive** `timestamp` and `retweeted_status_timestamp` - needs to be datetime for ease of analysis
* In **twitter_archive** `source` - retrieve the ahref tag contents only
* In **twitter_archive** `rating_numerator` and `rating_denominator` are to be cleaned and standardised. There are several instances when ratings were not gathered correctly ('24/7'  or 'Happy 4/20'), and where there are multiple ratings.
* In **image_predictions** there are missing values as 2075 observations compared to 2356 tweet IDs, suggesting some tweets are missing images, we may want to get rid of them.
* In **tweet_json** additional information from 8 tweets could not be obtained, probably because they were deleted on Twitter servers.
* In **image_predictions** dog breeds are of different case and spacing styles, replace all to lower case and replace '_' with regular spaces.
* Ensure all datatypes are appropriate.
***
## Clean


```python
# First and foremost - make copies
twitter_archive_copy = twitter_archive.copy()
image_predictions_copy = image_predictions.copy()
tweet_json_copy = tweet_json.copy()
```

### Tweet IDs to string format
In order to merge all three datasets together, firstly we must to ensure that all twitter IDs are to be string, as of [Twitter best practices](https://developer.twitter.com/en/docs/tweets/data-dictionary/overview/tweet-object)  
*Note:* we do not require to convert `in_reply_to_status_id`, `in_reply_to_user_id`, `retweeted_status_id`, `retweeted_status_user_id`, as these will only be used to identify retweets and then deleted.
#### Code


```python
twitter_archive_copy.tweet_id = twitter_archive_copy.tweet_id.astype('str')
image_predictions_copy.tweet_id = image_predictions_copy.tweet_id.astype('str')
tweet_json_copy.tweet_id = tweet_json_copy.tweet_id.astype('str')

# Check the outcome
print(twitter_archive_copy.tweet_id.dtype)
print(image_predictions_copy.tweet_id.dtype)
print(tweet_json_copy.tweet_id.dtype)
```

    object
    object
    object
    

### Combine all dataframes
All the dataframes should be combined into a single dataframe **dog_ratings**.
#### Code


```python
dog_ratings = pd.merge(twitter_archive_copy, image_predictions_copy, how = 'left', on = ['tweet_id'] )
dog_ratings = pd.merge(dog_ratings, tweet_json_copy, how = 'left', on = ['tweet_id'])
# Check the outcome
dog_ratings.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2356 entries, 0 to 2355
    Data columns (total 30 columns):
    tweet_id                      2356 non-null object
    in_reply_to_status_id         78 non-null float64
    in_reply_to_user_id           78 non-null float64
    timestamp                     2356 non-null object
    source                        2356 non-null object
    text                          2356 non-null object
    retweeted_status_id           181 non-null float64
    retweeted_status_user_id      181 non-null float64
    retweeted_status_timestamp    181 non-null object
    expanded_urls                 2297 non-null object
    rating_numerator              2356 non-null int64
    rating_denominator            2356 non-null int64
    name                          2356 non-null object
    doggo                         2356 non-null object
    floofer                       2356 non-null object
    pupper                        2356 non-null object
    puppo                         2356 non-null object
    jpg_url                       2075 non-null object
    img_num                       2075 non-null float64
    p1                            2075 non-null object
    p1_conf                       2075 non-null float64
    p1_dog                        2075 non-null object
    p2                            2075 non-null object
    p2_conf                       2075 non-null float64
    p2_dog                        2075 non-null object
    p3                            2075 non-null object
    p3_conf                       2075 non-null float64
    p3_dog                        2075 non-null object
    favorite_count                2345 non-null float64
    retweet_count                 2345 non-null float64
    dtypes: float64(10), int64(2), object(18)
    memory usage: 570.6+ KB
    

### Dog stage
Create a single categorical variable `dog_stage` and melt `doggo`, `floofer`, `pupper`, `puppo`.
#### Code


```python
# List of variables to keep after melt
id_vars_list = ['tweet_id', 'in_reply_to_status_id', 'in_reply_to_user_id', 'timestamp', 'source', 'text', 
                'retweeted_status_id', 'retweeted_status_user_id', 'retweeted_status_timestamp', 'expanded_urls', 
                'rating_numerator', 'rating_denominator', 'name', 'jpg_url', 'img_num', 'p1', 'p1_conf', 'p1_dog', 
                'p2', 'p2_conf', 'p2_dog', 'p3', 'p3_conf', 'p3_dog', 'favorite_count', 'retweet_count']

# Melt floofer, doggo, pupper, puppo  variables into a new column, 
# this will create a very long list of 4 instances per variable
dog_ratings =pd.melt(dog_ratings, id_vars = id_vars_list, var_name = 'dog_stage_temp', value_name = 'dog_stage')

# Remove the unnessesary column of column names
dog_ratings = dog_ratings.drop('dog_stage_temp', axis =1)

# Remove duplicated tweet ids and keep last instance of dog type, as all others are 'None'
dog_ratings = dog_ratings.sort_values('dog_stage').drop_duplicates('tweet_id', keep = 'last')

# Convert 'None' to NaN
dog_ratings['dog_stage'].replace('None', np.nan, inplace=True)

# Assign appropriate data type
dog_ratings.dog_stage = dog_ratings.dog_stage.astype('category')

# Check the outcome
dog_ratings.dog_stage.value_counts(dropna = False)
```




    NaN        1976
    pupper     257 
    doggo      83  
    puppo      30  
    floofer    10  
    Name: dog_stage, dtype: int64



### Dog Breeds variable
Create a new column called `dog_breed`, populate it with `p(n)` value, where `p(n)_conf` is highest and `p(n)_dog` is set to True. Thanks to amazing Udacity wizardry on image predictions' the confidence variables are listed in descending order, meaning that p1_conf is higher than p2_conf or p3_conf. This makes the logic of choosing the dog breed from predictions very simple.
#### Code


```python
# New list of dog breeds
dog_breed_list = []
# A much shorter and ellegant way of itterating through several columns than nested for loop.
def breed_to_list(df):
    """
    This funcion checks the values in p(n)_dog,
    if either of them is True it will append the value of
    corresponding dog breed prediction into the list.
    If all predictions are set to False, it will append a 
    NaN value to the list.
    """
    if df['p1_dog'] == True:
        dog_breed_list.append(df['p1'])
    elif df['p2_dog'] == True:
        dog_breed_list.append(df['p2'])
    elif df['p3_dog'] == True:
        dog_breed_list.append(df['p3'])
    else:
        dog_breed_list.append(np.nan)
    return dog_breed_list

dog_ratings.apply(breed_to_list, axis=1)

# Incorporate the list into the dataframe's new column
dog_ratings['dog_breed'] = dog_breed_list

# Check the outcome
dog_ratings.dog_breed.value_counts()
```




    golden_retriever                  173
    Labrador_retriever                113
    Pembroke                          96 
    Chihuahua                         95 
    pug                               65 
    toy_poodle                        52 
    chow                              51 
    Samoyed                           46 
    Pomeranian                        42 
    cocker_spaniel                    34 
    malamute                          34 
    French_bulldog                    32 
    Chesapeake_Bay_retriever          31 
    miniature_pinscher                26 
    Cardigan                          23 
    Staffordshire_bullterrier         22 
    Eskimo_dog                        22 
    beagle                            21 
    German_shepherd                   21 
    Siberian_husky                    20 
    Shih-Tzu                          20 
    kuvasz                            19 
    Rottweiler                        19 
    Maltese_dog                       19 
    Lakeland_terrier                  19 
    Shetland_sheepdog                 19 
    Italian_greyhound                 17 
    basset                            17 
    West_Highland_white_terrier       16 
    American_Staffordshire_terrier    16 
                                      .. 
    Weimaraner                        4  
    Welsh_springer_spaniel            4  
    Scottish_deerhound                4  
    Rhodesian_ridgeback               4  
    keeshond                          4  
    giant_schnauzer                   4  
    Saluki                            4  
    Irish_water_spaniel               3  
    cairn                             3  
    Brabancon_griffon                 3  
    komondor                          3  
    Greater_Swiss_Mountain_dog        3  
    toy_terrier                       3  
    Leonberg                          3  
    curly-coated_retriever            3  
    briard                            3  
    groenendael                       2  
    black-and-tan_coonhound           2  
    wire-haired_fox_terrier           2  
    Appenzeller                       2  
    Australian_terrier                2  
    Sussex_spaniel                    2  
    standard_schnauzer                1  
    Scotch_terrier                    1  
    silky_terrier                     1  
    clumber                           1  
    EntleBucher                       1  
    Irish_wolfhound                   1  
    Bouvier_des_Flandres              1  
    Japanese_spaniel                  1  
    Name: dog_breed, Length: 113, dtype: int64



### No Retweets
Only original tweets to be used, exclude retweets and replies. Retweets and replies are identified by `retweet_rating_status_id` and `in_reply_to_status_id` respectively, which are assigned to the tweet status object if it is a retweet or reply.
#### Code


```python
# Nullify the observations where there is a retweet_status_id or in_reply_to_status_id
dog_ratings = dog_ratings[pd.isnull(dog_ratings.retweeted_status_id)]
dog_ratings = dog_ratings[pd.isnull(dog_ratings.in_reply_to_status_id)]

# Remove unnessesary columns
columns_to_drop = ['retweeted_status_id','retweeted_status_user_id','retweeted_status_timestamp',
                  'in_reply_to_status_id', 'in_reply_to_user_id']
dog_ratings.drop(columns_to_drop, axis = 1, inplace = True)

# Check the outcome
dog_ratings.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2097 entries, 2261 to 7236
    Data columns (total 23 columns):
    tweet_id              2097 non-null object
    timestamp             2097 non-null object
    source                2097 non-null object
    text                  2097 non-null object
    expanded_urls         2094 non-null object
    rating_numerator      2097 non-null int64
    rating_denominator    2097 non-null int64
    name                  2097 non-null object
    jpg_url               1971 non-null object
    img_num               1971 non-null float64
    p1                    1971 non-null object
    p1_conf               1971 non-null float64
    p1_dog                1971 non-null object
    p2                    1971 non-null object
    p2_conf               1971 non-null float64
    p2_dog                1971 non-null object
    p3                    1971 non-null object
    p3_conf               1971 non-null float64
    p3_dog                1971 non-null object
    favorite_count        2097 non-null float64
    retweet_count         2097 non-null float64
    dog_stage             336 non-null category
    dog_breed             1666 non-null object
    dtypes: category(1), float64(6), int64(2), object(14)
    memory usage: 379.0+ KB
    

### Remove tweets without images 
After all datasets were combined together we can match and remove all entries which do not have contents in `jpg_url` column, ideally the number of entries should match the size of **image_redictions**.
#### Code


```python
# Remove observations where there are NaN values in 'jpg_url' column
dog_ratings = dog_ratings.dropna(subset=['jpg_url'])
# Check the outcome
dog_ratings.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1971 entries, 2261 to 7236
    Data columns (total 23 columns):
    tweet_id              1971 non-null object
    timestamp             1971 non-null object
    source                1971 non-null object
    text                  1971 non-null object
    expanded_urls         1971 non-null object
    rating_numerator      1971 non-null int64
    rating_denominator    1971 non-null int64
    name                  1971 non-null object
    jpg_url               1971 non-null object
    img_num               1971 non-null float64
    p1                    1971 non-null object
    p1_conf               1971 non-null float64
    p1_dog                1971 non-null object
    p2                    1971 non-null object
    p2_conf               1971 non-null float64
    p2_dog                1971 non-null object
    p3                    1971 non-null object
    p3_conf               1971 non-null float64
    p3_dog                1971 non-null object
    favorite_count        1971 non-null float64
    retweet_count         1971 non-null float64
    dog_stage             303 non-null category
    dog_breed             1666 non-null object
    dtypes: category(1), float64(6), int64(2), object(14)
    memory usage: 356.3+ KB
    

### Tweet source
`source` variable is difficult to read, as it is an HTML tag, retrieve the href tag contents only.
#### Code


```python
# Check which values to extract
dog_ratings.source.value_counts()
```




    <a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>     1932
    <a href="http://twitter.com" rel="nofollow">Twitter Web Client</a>                     28  
    <a href="https://about.twitter.com/products/tweetdeck" rel="nofollow">TweetDeck</a>    11  
    Name: source, dtype: int64




```python
# Use regulal expressions to extract contents from source and replace it
dog_ratings['source'] = dog_ratings.source.str.extract('<a[^>]*>([^<]*)</a>', expand=True)
# Assign appripriate category
dog_ratings.source = dog_ratings.source.astype('category')
# Check the outcome
dog_ratings.source.value_counts()
```




    Twitter for iPhone    1932
    Twitter Web Client    28  
    TweetDeck             11  
    Name: source, dtype: int64



### Invalid rating
There are number of ratings numerators which go up to 100s/10. Denominators are mostly 10s, with few exceptions where there are multiple dogs involved in a tweet. Overall there are only 18 ratings not conforming with the standard.
The following wrangle will be in several steps as there are a lot of thisng to tackle.
1. There are 5 instances where the ratings are confused with other fractions such as 7/11, 9/11, 20/4, 24/7 etc, reading through the tweet text in Assessment part of this report, it was confirmed that these instances are not ratings and include actual rating further down the tweet, might as well go about this manually.
2. Remove observation with tweet_ids *670842764863651840 and 749981277374128128*, these are super high ratings which will skew the data. Also remove observation with tweet_id *810984652412424192*, as there is no rating involved.
3. Come up with a new rating system, perhaps only leaving rating numerator since denominatora are all 10 or a multiple of 10.
#### Code


```python
# 1 # 
# Separate instances where the denominator does not equal to 10
print('Total number of instances: ', len(dog_ratings.loc[dog_ratings.rating_denominator!=10,
                              ['tweet_id','text','rating_numerator','rating_denominator']]))
dog_ratings.loc[dog_ratings.rating_denominator!=10,
                              ['tweet_id','text','rating_numerator','rating_denominator']]
```

    Total number of instances:  17
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>text</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2335</th>
      <td>666287406224695296</td>
      <td>This is an Albanian 3 1/2 legged  Episcopalian. Loves well-polished hardwood flooring. Penis on the collar. 9/10 https://t.co/d9NcXFKwLv</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3789</th>
      <td>697463031882764288</td>
      <td>Happy Wednesday here's a bucket of pups. 44/40 would pet all at once https://t.co/HppvrYuamZ</td>
      <td>44</td>
      <td>40</td>
    </tr>
    <tr>
      <th>3707</th>
      <td>704054845121142784</td>
      <td>Here is a whole flock of puppers.  60/50 I'll take the lot https://t.co/9dpcw6MdWa</td>
      <td>60</td>
      <td>50</td>
    </tr>
    <tr>
      <th>3991</th>
      <td>684222868335505415</td>
      <td>Someone help the girl is being mugged. Several are distracting her while two steal her shoes. Clever puppers 121/110 https://t.co/1zfnTJLt55</td>
      <td>121</td>
      <td>110</td>
    </tr>
    <tr>
      <th>3521</th>
      <td>722974582966214656</td>
      <td>Happy 4/20 from the squad! 13/10 for all https://t.co/eV1diwds8a</td>
      <td>4</td>
      <td>20</td>
    </tr>
    <tr>
      <th>3558</th>
      <td>716439118184652801</td>
      <td>This is Bluebert. He just saw that both #FinalFur match ups are split 50/50. Amazed af. 11/10 https://t.co/Kky1DPG4iq</td>
      <td>50</td>
      <td>50</td>
    </tr>
    <tr>
      <th>3424</th>
      <td>740373189193256964</td>
      <td>After so many requests, this is Bretagne. She was the last surviving 9/11 search dog, and our second ever 14/10. RIP https://t.co/XAVDNDaVgQ</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>3476</th>
      <td>731156023742988288</td>
      <td>Say hello to this unbelievably well behaved squad of doggos. 204/170 would try to pet all at once https://t.co/yGQI3He3xv</td>
      <td>204</td>
      <td>170</td>
    </tr>
    <tr>
      <th>3584</th>
      <td>713900603437621249</td>
      <td>Happy Saturday here's 9 puppers on a bench. 99/90 good work everybody https://t.co/mpvaVxKmc1</td>
      <td>99</td>
      <td>90</td>
    </tr>
    <tr>
      <th>3630</th>
      <td>709198395643068416</td>
      <td>From left to right:\nCletus, Jerome, Alejandro, Burp, &amp;amp; Titson\nNone know where camera is. 45/50 would hug all at once https://t.co/sedre1ivTK</td>
      <td>45</td>
      <td>50</td>
    </tr>
    <tr>
      <th>3610</th>
      <td>710658690886586372</td>
      <td>Here's a brigade of puppers. All look very prepared for whatever happens next. 80/80 https://t.co/0eb7R1Om12</td>
      <td>80</td>
      <td>80</td>
    </tr>
    <tr>
      <th>4018</th>
      <td>682962037429899265</td>
      <td>This is Darrel. He just robbed a 7/11 and is in a high speed police chase. Was just spotted by the helicopter 10/10 https://t.co/7EsP8LmSp5</td>
      <td>7</td>
      <td>11</td>
    </tr>
    <tr>
      <th>4135</th>
      <td>677716515794329600</td>
      <td>IT'S PUPPERGEDDON. Total of 144/120 ...I think https://t.co/ZanVtAtvIq</td>
      <td>144</td>
      <td>120</td>
    </tr>
    <tr>
      <th>4199</th>
      <td>675853064436391936</td>
      <td>Here we have an entire platoon of puppers. Total score: 88/80 would pet all at once https://t.co/y93p6FLvVw</td>
      <td>88</td>
      <td>80</td>
    </tr>
    <tr>
      <th>2872</th>
      <td>810984652412424192</td>
      <td>Meet Sam. She smiles 24/7 &amp;amp; secretly aspires to be a reindeer. \nKeep Sam smiling by clicking and sharing this link:\nhttps://t.co/98tB8y7y7t https://t.co/LouL5vdvxx</td>
      <td>24</td>
      <td>7</td>
    </tr>
    <tr>
      <th>2789</th>
      <td>820690176645140481</td>
      <td>The floofs have been released I repeat the floofs have been released. 84/70 https://t.co/NIYC820tmd</td>
      <td>84</td>
      <td>70</td>
    </tr>
    <tr>
      <th>3258</th>
      <td>758467244762497024</td>
      <td>Why does this never happen at my front door... 165/150 https://t.co/HmwrdfEfUE</td>
      <td>165</td>
      <td>150</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 1 # Locate tweet_ids with confused ratings and correct them, 5 in total.
dog_ratings.loc[dog_ratings.tweet_id == '666287406224695296','rating_numerator'] = 9
dog_ratings.loc[dog_ratings.tweet_id == '722974582966214656','rating_numerator'] = 13
dog_ratings.loc[dog_ratings.tweet_id == '716439118184652801','rating_numerator'] = 11
dog_ratings.loc[dog_ratings.tweet_id == '740373189193256964','rating_numerator'] = 14
dog_ratings.loc[dog_ratings.tweet_id == '682962037429899265','rating_numerator'] = 10
```


```python
# 2 # Remove three observations ratings for which are outliers
dog_ratings = dog_ratings[dog_ratings.tweet_id!='810984652412424192']
dog_ratings = dog_ratings[dog_ratings.tweet_id!='670842764863651840']
dog_ratings = dog_ratings[dog_ratings.tweet_id!='749981277374128128']
```


```python
# Check the outcome
dog_ratings.rating_numerator.value_counts()
```




    12     446
    10     418
    11     393
    13     254
    9      150
    8      95 
    7      51 
    14     34 
    5      33 
    6      32 
    3      19 
    4      15 
    2      9  
    1      4  
    204    1  
    165    1  
    26     1  
    27     1  
    44     1  
    45     1  
    60     1  
    75     1  
    80     1  
    84     1  
    88     1  
    99     1  
    121    1  
    144    1  
    0      1  
    Name: rating_numerator, dtype: int64



### Make dog breeds more readable
Make `dog_breed` more uniform by replacing underscore with space and turning all instances to lower case.


```python
dog_ratings.dog_breed = dog_ratings.dog_breed.str.replace('_',' ')
dog_ratings.dog_breed = dog_ratings.dog_breed.str.lower()
```

#### Final Clean-up and data type corrections
Final clean dataframe **dog_ratings** to include following columns:
* `tweet_id` as obj
* `source` as cat
* `timestamp` as datetime
* `rating_numerator` as int
* `rating_denominator` as int
* `img_url` as obj
* `dog_breed` as cat
* `dog_stage` as cat
* `favorite_count` as int
* `retweet_count` as int
#### Code


```python
dog_ratings.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1968 entries, 2261 to 7236
    Data columns (total 23 columns):
    tweet_id              1968 non-null object
    timestamp             1968 non-null object
    source                1968 non-null category
    text                  1968 non-null object
    expanded_urls         1968 non-null object
    rating_numerator      1968 non-null int64
    rating_denominator    1968 non-null int64
    name                  1968 non-null object
    jpg_url               1968 non-null object
    img_num               1968 non-null float64
    p1                    1968 non-null object
    p1_conf               1968 non-null float64
    p1_dog                1968 non-null object
    p2                    1968 non-null object
    p2_conf               1968 non-null float64
    p2_dog                1968 non-null object
    p3                    1968 non-null object
    p3_conf               1968 non-null float64
    p3_dog                1968 non-null object
    favorite_count        1968 non-null float64
    retweet_count         1968 non-null float64
    dog_stage             303 non-null category
    dog_breed             1665 non-null object
    dtypes: category(2), float64(6), int64(2), object(13)
    memory usage: 342.4+ KB
    


```python
# Dropping unnessesary columns
columns_to_drop = ['text','expanded_urls','rating_denominator','name','img_num',
                   'p1','p1_conf','p1_dog','p2','p2_conf','p2_dog','p3','p3_conf','p3_dog']
dog_ratings.drop(columns_to_drop, axis = 1, inplace = True)
# Check the outcome
dog_ratings.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1968 entries, 2261 to 7236
    Data columns (total 9 columns):
    tweet_id            1968 non-null object
    timestamp           1968 non-null object
    source              1968 non-null category
    rating_numerator    1968 non-null int64
    jpg_url             1968 non-null object
    favorite_count      1968 non-null float64
    retweet_count       1968 non-null float64
    dog_stage           303 non-null category
    dog_breed           1665 non-null object
    dtypes: category(2), float64(2), int64(1), object(4)
    memory usage: 127.1+ KB
    


```python
# Assign shorter column names
dog_ratings = dog_ratings.rename(columns={'dog_breed' : 'breed',
                                          'dog_stage' : 'stage',
                                          'rating_numerator' : 'rating'})
# Fixing data types
dog_ratings.timestamp = pd.to_datetime(dog_ratings.timestamp)
dog_ratings.source = dog_ratings.source.astype('category')
dog_ratings.favorite_count = dog_ratings.favorite_count.astype(int)
dog_ratings.retweet_count = dog_ratings.retweet_count.astype(int)
dog_ratings.breed = dog_ratings.breed.astype('category')
dog_ratings.rating = dog_ratings.rating.astype(int)

# Check the outcome
dog_ratings.dtypes
```




    tweet_id          object        
    timestamp         datetime64[ns]
    source            category      
    rating            int32         
    jpg_url           object        
    favorite_count    int32         
    retweet_count     int32         
    stage             category      
    breed             category      
    dtype: object



## Saving the data


```python
# Final look at the final clean dataset
dog_ratings
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>rating</th>
      <th>jpg_url</th>
      <th>favorite_count</th>
      <th>retweet_count</th>
      <th>stage</th>
      <th>breed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2261</th>
      <td>667549055577362432</td>
      <td>2015-11-20 03:44:31</td>
      <td>Twitter Web Client</td>
      <td>1</td>
      <td>https://pbs.twimg.com/media/CUOcVCwWsAERUKY.jpg</td>
      <td>5977</td>
      <td>2393</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2262</th>
      <td>667546741521195010</td>
      <td>2015-11-20 03:35:20</td>
      <td>Twitter Web Client</td>
      <td>9</td>
      <td>https://pbs.twimg.com/media/CUOaOWXWcAA0_Jy.jpg</td>
      <td>342</td>
      <td>132</td>
      <td>NaN</td>
      <td>toy poodle</td>
    </tr>
    <tr>
      <th>2263</th>
      <td>667544320556335104</td>
      <td>2015-11-20 03:25:43</td>
      <td>Twitter Web Client</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/CUOYBbbWIAAXQGU.jpg</td>
      <td>895</td>
      <td>549</td>
      <td>NaN</td>
      <td>pomeranian</td>
    </tr>
    <tr>
      <th>2264</th>
      <td>667538891197542400</td>
      <td>2015-11-20 03:04:08</td>
      <td>Twitter Web Client</td>
      <td>9</td>
      <td>https://pbs.twimg.com/media/CUOTFZOW4AABsfW.jpg</td>
      <td>209</td>
      <td>70</td>
      <td>NaN</td>
      <td>yorkshire terrier</td>
    </tr>
    <tr>
      <th>2258</th>
      <td>667724302356258817</td>
      <td>2015-11-20 15:20:54</td>
      <td>Twitter Web Client</td>
      <td>7</td>
      <td>https://pbs.twimg.com/media/CUQ7tv3W4AA3KlI.jpg</td>
      <td>503</td>
      <td>332</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2265</th>
      <td>667534815156183040</td>
      <td>2015-11-20 02:47:56</td>
      <td>Twitter Web Client</td>
      <td>8</td>
      <td>https://pbs.twimg.com/media/CUOPYI5UcAAj_nO.jpg</td>
      <td>844</td>
      <td>561</td>
      <td>NaN</td>
      <td>pembroke</td>
    </tr>
    <tr>
      <th>2267</th>
      <td>667524857454854144</td>
      <td>2015-11-20 02:08:22</td>
      <td>Twitter Web Client</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/CUOGUfJW4AA_eni.jpg</td>
      <td>1751</td>
      <td>1167</td>
      <td>NaN</td>
      <td>chesapeake bay retriever</td>
    </tr>
    <tr>
      <th>2268</th>
      <td>667517642048163840</td>
      <td>2015-11-20 01:39:42</td>
      <td>Twitter Web Client</td>
      <td>8</td>
      <td>https://pbs.twimg.com/media/CUN_wiBUkAAakT0.jpg</td>
      <td>378</td>
      <td>198</td>
      <td>NaN</td>
      <td>italian greyhound</td>
    </tr>
    <tr>
      <th>2269</th>
      <td>667509364010450944</td>
      <td>2015-11-20 01:06:48</td>
      <td>Twitter Web Client</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/CUN4Or5UAAAa5K4.jpg</td>
      <td>6986</td>
      <td>2215</td>
      <td>NaN</td>
      <td>beagle</td>
    </tr>
    <tr>
      <th>2270</th>
      <td>667502640335572993</td>
      <td>2015-11-20 00:40:05</td>
      <td>Twitter Web Client</td>
      <td>11</td>
      <td>https://pbs.twimg.com/media/CUNyHTMUYAAQVch.jpg</td>
      <td>545</td>
      <td>227</td>
      <td>NaN</td>
      <td>labrador retriever</td>
    </tr>
    <tr>
      <th>2271</th>
      <td>667495797102141441</td>
      <td>2015-11-20 00:12:54</td>
      <td>Twitter Web Client</td>
      <td>9</td>
      <td>https://pbs.twimg.com/media/CUNr4-7UwAAg2lq.jpg</td>
      <td>544</td>
      <td>287</td>
      <td>NaN</td>
      <td>chihuahua</td>
    </tr>
    <tr>
      <th>2272</th>
      <td>667491009379606528</td>
      <td>2015-11-19 23:53:52</td>
      <td>Twitter Web Client</td>
      <td>7</td>
      <td>https://pbs.twimg.com/media/CUNniSlUYAEj1Jl.jpg</td>
      <td>548</td>
      <td>237</td>
      <td>NaN</td>
      <td>borzoi</td>
    </tr>
    <tr>
      <th>2266</th>
      <td>667530908589760512</td>
      <td>2015-11-20 02:32:25</td>
      <td>Twitter Web Client</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/CUOL0uGUkAAx7yh.jpg</td>
      <td>488</td>
      <td>254</td>
      <td>NaN</td>
      <td>golden retriever</td>
    </tr>
    <tr>
      <th>2273</th>
      <td>667470559035432960</td>
      <td>2015-11-19 22:32:36</td>
      <td>Twitter Web Client</td>
      <td>11</td>
      <td>https://pbs.twimg.com/media/CUNU78YWEAECmpB.jpg</td>
      <td>263</td>
      <td>101</td>
      <td>NaN</td>
      <td>toy poodle</td>
    </tr>
    <tr>
      <th>2257</th>
      <td>667728196545200128</td>
      <td>2015-11-20 15:36:22</td>
      <td>Twitter Web Client</td>
      <td>11</td>
      <td>https://pbs.twimg.com/media/CUQ_QahUAAAVQjn.jpg</td>
      <td>384</td>
      <td>158</td>
      <td>NaN</td>
      <td>kuvasz</td>
    </tr>
    <tr>
      <th>2255</th>
      <td>667773195014021121</td>
      <td>2015-11-20 18:35:10</td>
      <td>Twitter Web Client</td>
      <td>8</td>
      <td>https://pbs.twimg.com/media/CURoLrOVEAAaWdR.jpg</td>
      <td>240</td>
      <td>59</td>
      <td>NaN</td>
      <td>west highland white terrier</td>
    </tr>
    <tr>
      <th>2241</th>
      <td>667915453470232577</td>
      <td>2015-11-21 04:00:28</td>
      <td>Twitter for iPhone</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/CUTpj-GWcAATc6A.jpg</td>
      <td>217</td>
      <td>58</td>
      <td>NaN</td>
      <td>boxer</td>
    </tr>
    <tr>
      <th>2242</th>
      <td>667911425562669056</td>
      <td>2015-11-21 03:44:27</td>
      <td>Twitter for iPhone</td>
      <td>5</td>
      <td>https://pbs.twimg.com/media/CUTl5m1WUAAabZG.jpg</td>
      <td>511</td>
      <td>318</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2243</th>
      <td>667902449697558528</td>
      <td>2015-11-21 03:08:47</td>
      <td>Twitter for iPhone</td>
      <td>9</td>
      <td>https://pbs.twimg.com/media/CUTdvAJXIAAMS4q.jpg</td>
      <td>876</td>
      <td>390</td>
      <td>NaN</td>
      <td>norwegian elkhound</td>
    </tr>
    <tr>
      <th>2244</th>
      <td>667886921285246976</td>
      <td>2015-11-21 02:07:05</td>
      <td>Twitter for iPhone</td>
      <td>11</td>
      <td>https://pbs.twimg.com/media/CUTPnPCW4AI7R0y.jpg</td>
      <td>1967</td>
      <td>1149</td>
      <td>NaN</td>
      <td>pomeranian</td>
    </tr>
    <tr>
      <th>2245</th>
      <td>667885044254572545</td>
      <td>2015-11-21 01:59:37</td>
      <td>Twitter for iPhone</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/CUTN5V4XAAAIa4R.jpg</td>
      <td>846</td>
      <td>508</td>
      <td>NaN</td>
      <td>malamute</td>
    </tr>
    <tr>
      <th>2246</th>
      <td>667878741721415682</td>
      <td>2015-11-21 01:34:35</td>
      <td>Twitter for iPhone</td>
      <td>2</td>
      <td>https://pbs.twimg.com/media/CUTILFiWcAE8Rle.jpg</td>
      <td>402</td>
      <td>124</td>
      <td>NaN</td>
      <td>miniature pinscher</td>
    </tr>
    <tr>
      <th>2256</th>
      <td>667766675769573376</td>
      <td>2015-11-20 18:09:16</td>
      <td>Twitter Web Client</td>
      <td>9</td>
      <td>https://pbs.twimg.com/media/CURiQMnUAAAPT2M.jpg</td>
      <td>461</td>
      <td>229</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2247</th>
      <td>667873844930215936</td>
      <td>2015-11-21 01:15:07</td>
      <td>Twitter for iPhone</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/CUTDtyGXIAARxus.jpg</td>
      <td>651</td>
      <td>427</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2249</th>
      <td>667861340749471744</td>
      <td>2015-11-21 00:25:26</td>
      <td>Twitter for iPhone</td>
      <td>9</td>
      <td>https://pbs.twimg.com/media/CUS4WJ-UsAEJj10.jpg</td>
      <td>250</td>
      <td>81</td>
      <td>NaN</td>
      <td>malamute</td>
    </tr>
    <tr>
      <th>2250</th>
      <td>667832474953625600</td>
      <td>2015-11-20 22:30:44</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/CUSeGFNW4AAyyHC.jpg</td>
      <td>294</td>
      <td>65</td>
      <td>NaN</td>
      <td>miniature pinscher</td>
    </tr>
    <tr>
      <th>2251</th>
      <td>667806454573760512</td>
      <td>2015-11-20 20:47:20</td>
      <td>Twitter for iPhone</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/CUSGbXeVAAAgztZ.jpg</td>
      <td>1091</td>
      <td>524</td>
      <td>NaN</td>
      <td>chihuahua</td>
    </tr>
    <tr>
      <th>2252</th>
      <td>667801013445750784</td>
      <td>2015-11-20 20:25:43</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/CUSBemVUEAAn-6V.jpg</td>
      <td>338</td>
      <td>98</td>
      <td>NaN</td>
      <td>flat-coated retriever</td>
    </tr>
    <tr>
      <th>2253</th>
      <td>667793409583771648</td>
      <td>2015-11-20 19:55:30</td>
      <td>Twitter for iPhone</td>
      <td>8</td>
      <td>https://pbs.twimg.com/media/CUR6jqVWsAEgGot.jpg</td>
      <td>719</td>
      <td>343</td>
      <td>NaN</td>
      <td>dalmatian</td>
    </tr>
    <tr>
      <th>2254</th>
      <td>667782464991965184</td>
      <td>2015-11-20 19:12:01</td>
      <td>Twitter for iPhone</td>
      <td>9</td>
      <td>https://pbs.twimg.com/media/CURwm3cUkAARcO6.jpg</td>
      <td>425</td>
      <td>253</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>5501</th>
      <td>773985732834758656</td>
      <td>2016-09-08 20:45:53</td>
      <td>Twitter for iPhone</td>
      <td>11</td>
      <td>https://pbs.twimg.com/media/Cr2_6R8WAAAUMtc.jpg</td>
      <td>11686</td>
      <td>4352</td>
      <td>pupper</td>
      <td>pug</td>
    </tr>
    <tr>
      <th>6557</th>
      <td>675845657354215424</td>
      <td>2015-12-13 01:12:15</td>
      <td>Twitter for iPhone</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/CWEWClfW4AAnqhG.jpg</td>
      <td>2408</td>
      <td>966</td>
      <td>pupper</td>
      <td>pug</td>
    </tr>
    <tr>
      <th>6091</th>
      <td>701545186879471618</td>
      <td>2016-02-21 23:13:01</td>
      <td>Twitter for iPhone</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/CbxjnyOWAAAWLUH.jpg</td>
      <td>2838</td>
      <td>664</td>
      <td>pupper</td>
      <td>border collie</td>
    </tr>
    <tr>
      <th>6562</th>
      <td>675740360753160193</td>
      <td>2015-12-12 18:13:51</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/ext_tw_video_thumb/675740268751138818/pu/img/dVaVeFAVT-lk_1ZV.jpg</td>
      <td>1224</td>
      <td>373</td>
      <td>pupper</td>
      <td>golden retriever</td>
    </tr>
    <tr>
      <th>4961</th>
      <td>845306882940190720</td>
      <td>2017-03-24 16:10:40</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/C7siH5DXkAACnDT.jpg</td>
      <td>24737</td>
      <td>5868</td>
      <td>pupper</td>
      <td>irish water spaniel</td>
    </tr>
    <tr>
      <th>6072</th>
      <td>703268521220972544</td>
      <td>2016-02-26 17:20:56</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/CcKC-5LW4AAK-nb.jpg</td>
      <td>2108</td>
      <td>607</td>
      <td>pupper</td>
      <td>kuvasz</td>
    </tr>
    <tr>
      <th>5488</th>
      <td>776113305656188928</td>
      <td>2016-09-14 17:40:06</td>
      <td>Twitter for iPhone</td>
      <td>11</td>
      <td>https://pbs.twimg.com/media/CsVO7ljW8AAckRD.jpg</td>
      <td>12798</td>
      <td>4891</td>
      <td>pupper</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7711</th>
      <td>793195938047070209</td>
      <td>2016-10-31 21:00:23</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/CwH_foYWgAEvTyI.jpg</td>
      <td>16724</td>
      <td>6342</td>
      <td>puppo</td>
      <td>labrador retriever</td>
    </tr>
    <tr>
      <th>7082</th>
      <td>889531135344209921</td>
      <td>2017-07-24 17:02:04</td>
      <td>Twitter for iPhone</td>
      <td>13</td>
      <td>https://pbs.twimg.com/media/DFg_2PVW0AEHN3p.jpg</td>
      <td>15108</td>
      <td>2252</td>
      <td>puppo</td>
      <td>golden retriever</td>
    </tr>
    <tr>
      <th>8029</th>
      <td>751132876104687617</td>
      <td>2016-07-07 19:16:47</td>
      <td>Twitter for iPhone</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/CmyPXNOW8AEtaJ-.jpg</td>
      <td>5497</td>
      <td>1441</td>
      <td>puppo</td>
      <td>labrador retriever</td>
    </tr>
    <tr>
      <th>7162</th>
      <td>874012996292530176</td>
      <td>2017-06-11 21:18:31</td>
      <td>Twitter for iPhone</td>
      <td>13</td>
      <td>https://pbs.twimg.com/media/DCEeLxjXsAAvNSM.jpg</td>
      <td>34801</td>
      <td>10635</td>
      <td>puppo</td>
      <td>cardigan</td>
    </tr>
    <tr>
      <th>8151</th>
      <td>738537504001953792</td>
      <td>2016-06-03 01:07:16</td>
      <td>Twitter for iPhone</td>
      <td>11</td>
      <td>https://pbs.twimg.com/media/Cj_P7rSUgAAYQbz.jpg</td>
      <td>5459</td>
      <td>1714</td>
      <td>puppo</td>
      <td>chow</td>
    </tr>
    <tr>
      <th>7757</th>
      <td>787717603741622272</td>
      <td>2016-10-16 18:11:26</td>
      <td>Twitter for iPhone</td>
      <td>13</td>
      <td>https://pbs.twimg.com/media/Cu6I9vvWIAAZG0a.jpg</td>
      <td>11173</td>
      <td>3133</td>
      <td>puppo</td>
      <td>german shepherd</td>
    </tr>
    <tr>
      <th>8116</th>
      <td>743253157753532416</td>
      <td>2016-06-16 01:25:36</td>
      <td>Twitter for iPhone</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/ClCQzFUUYAA5vAu.jpg</td>
      <td>4531</td>
      <td>1330</td>
      <td>puppo</td>
      <td>malamute</td>
    </tr>
    <tr>
      <th>7731</th>
      <td>790946055508652032</td>
      <td>2016-10-25 16:00:09</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/CvoBPWRWgAA4het.jpg</td>
      <td>18184</td>
      <td>5312</td>
      <td>puppo</td>
      <td>golden retriever</td>
    </tr>
    <tr>
      <th>7990</th>
      <td>756275833623502848</td>
      <td>2016-07-21 23:53:04</td>
      <td>Twitter for iPhone</td>
      <td>10</td>
      <td>https://pbs.twimg.com/media/Cn7U2xlW8AI9Pqp.jpg</td>
      <td>6965</td>
      <td>1698</td>
      <td>puppo</td>
      <td>airedale</td>
    </tr>
    <tr>
      <th>7635</th>
      <td>802239329049477120</td>
      <td>2016-11-25 19:55:35</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/CyIgaTEVEAA-9zS.jpg</td>
      <td>9908</td>
      <td>2936</td>
      <td>puppo</td>
      <td>eskimo dog</td>
    </tr>
    <tr>
      <th>7139</th>
      <td>878776093423087618</td>
      <td>2017-06-25 00:45:22</td>
      <td>Twitter for iPhone</td>
      <td>13</td>
      <td>https://pbs.twimg.com/media/DDIKMXzW0AEibje.jpg</td>
      <td>19395</td>
      <td>4167</td>
      <td>puppo</td>
      <td>italian greyhound</td>
    </tr>
    <tr>
      <th>7507</th>
      <td>819952236453363712</td>
      <td>2017-01-13 17:00:21</td>
      <td>Twitter for iPhone</td>
      <td>13</td>
      <td>https://pbs.twimg.com/media/C2EONHNWQAUWxkP.jpg</td>
      <td>5784</td>
      <td>1325</td>
      <td>puppo</td>
      <td>american staffordshire terrier</td>
    </tr>
    <tr>
      <th>7481</th>
      <td>822872901745569793</td>
      <td>2017-01-21 18:26:02</td>
      <td>Twitter for iPhone</td>
      <td>13</td>
      <td>https://pbs.twimg.com/media/C2tugXLXgAArJO4.jpg</td>
      <td>143466</td>
      <td>49153</td>
      <td>puppo</td>
      <td>lakeland terrier</td>
    </tr>
    <tr>
      <th>7080</th>
      <td>889665388333682689</td>
      <td>2017-07-25 01:55:32</td>
      <td>Twitter for iPhone</td>
      <td>13</td>
      <td>https://pbs.twimg.com/media/DFi579UWsAAatzw.jpg</td>
      <td>48159</td>
      <td>10141</td>
      <td>puppo</td>
      <td>pembroke</td>
    </tr>
    <tr>
      <th>7804</th>
      <td>780931614150983680</td>
      <td>2016-09-28 00:46:20</td>
      <td>Twitter for iPhone</td>
      <td>13</td>
      <td>https://pbs.twimg.com/media/CtZtJxAXEAAyPGd.jpg</td>
      <td>23600</td>
      <td>8273</td>
      <td>puppo</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7259</th>
      <td>855851453814013952</td>
      <td>2017-04-22 18:31:02</td>
      <td>Twitter for iPhone</td>
      <td>13</td>
      <td>https://pbs.twimg.com/media/C-CYWrvWAAU8AXH.jpg</td>
      <td>46984</td>
      <td>18700</td>
      <td>puppo</td>
      <td>flat-coated retriever</td>
    </tr>
    <tr>
      <th>8015</th>
      <td>752519690950500352</td>
      <td>2016-07-11 15:07:30</td>
      <td>Twitter for iPhone</td>
      <td>11</td>
      <td>https://pbs.twimg.com/media/CnF8qVDWYAAh0g1.jpg</td>
      <td>7969</td>
      <td>3809</td>
      <td>puppo</td>
      <td>labrador retriever</td>
    </tr>
    <tr>
      <th>7197</th>
      <td>867421006826221569</td>
      <td>2017-05-24 16:44:18</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/DAmyy8FXYAIH8Ty.jpg</td>
      <td>16403</td>
      <td>2594</td>
      <td>puppo</td>
      <td>eskimo dog</td>
    </tr>
    <tr>
      <th>8103</th>
      <td>744995568523612160</td>
      <td>2016-06-20 20:49:19</td>
      <td>Twitter for iPhone</td>
      <td>9</td>
      <td>https://pbs.twimg.com/media/ClbBg4WWEAMjwJu.jpg</td>
      <td>3208</td>
      <td>692</td>
      <td>puppo</td>
      <td>old english sheepdog</td>
    </tr>
    <tr>
      <th>7463</th>
      <td>825535076884762624</td>
      <td>2017-01-29 02:44:34</td>
      <td>Twitter for iPhone</td>
      <td>14</td>
      <td>https://pbs.twimg.com/media/C3TjvitXAAAI-QH.jpg</td>
      <td>56217</td>
      <td>19197</td>
      <td>puppo</td>
      <td>rottweiler</td>
    </tr>
    <tr>
      <th>7466</th>
      <td>825026590719483904</td>
      <td>2017-01-27 17:04:02</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/C3MVTeHWcAAGNfx.jpg</td>
      <td>6875</td>
      <td>1440</td>
      <td>puppo</td>
      <td>eskimo dog</td>
    </tr>
    <tr>
      <th>7622</th>
      <td>803773340896923648</td>
      <td>2016-11-30 01:31:12</td>
      <td>Twitter for iPhone</td>
      <td>12</td>
      <td>https://pbs.twimg.com/media/CyeTku-XcAALkBd.jpg</td>
      <td>10985</td>
      <td>3124</td>
      <td>puppo</td>
      <td>miniature pinscher</td>
    </tr>
    <tr>
      <th>7236</th>
      <td>859607811541651456</td>
      <td>2017-05-03 03:17:27</td>
      <td>Twitter for iPhone</td>
      <td>13</td>
      <td>https://pbs.twimg.com/media/C-3wvtxXcAUTuBE.jpg</td>
      <td>19108</td>
      <td>1650</td>
      <td>puppo</td>
      <td>golden retriever</td>
    </tr>
  </tbody>
</table>
<p>1968 rows × 9 columns</p>
</div>




```python
# Save the dataframe to csv
dog_ratings.to_csv('twitter_archive_master.csv', index=False, encoding = 'utf-8')
```

## Resources
[Tweepy Documentation](https://media.readthedocs.org/pdf/tweepy/latest/tweepy.pdf)  
[Structure of Tweet JSON](https://developer.twitter.com/en/docs/tweets/data-dictionary/overview/intro-to-tweet-json)  
[Pandas Melt](https://www.youtube.com/watch?v=oY62o-tBHF4)  
All code is based on course examples provided by Udacity. One helpful suggestione was talen from Udacity Student Forum community regarding importing the additional twitter via the API.
#### Report created and compiled by Alina Bolat for Udacity Data Analytics Nano Degree
