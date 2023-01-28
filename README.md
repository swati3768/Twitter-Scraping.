import datetime
import pandas as pd
import snscrape.modules.twitter as sntwitter
import pymongo
import datetime
import json
import streamlit as st


header = st.container()
st.set_page_config(page_title='Twitter data',page_icon=":wave:",layout="wide")
st.title(" download all your tweets data from Twitter")
st.subheader("Twitter data downloader")
st.text("You will download information such as date, id, url, tweet content, user,reply count, retweet count,language, source, like count etc from twitter")


user_name = st.text_input('Plz write UserName or HashTag')
st.write('The scraps are related to ', user_name)
option = st.selectbox('How many scraps do you want?',
    return int(input("Enter a number between o and 1000, inclusive: "))
st.write('You selected:', option)
def inclusive():
    i = -1
    while i < 0 or i > 1000:
        print("Out of range. Try again!")
        i = option
    return i


a = datetime.datetime.today()
numdays = 100
dateList = []
for x in range (0, numdays):
    dateList.append(a - datetime.timedelta(days = x))

# Creating list to append tweet data 
tweets_list1 = []

# Using TwitterSearchScraper to scrape data and append tweets to list
for i,tweet in enumerate(sntwitter.TwitterSearchScraper(username).get_items()):
    if i>(int(option)-1):
        break
    else:
        tweets_list1.append([tweet.date, tweet.id, tweet.url, tweet.content, tweet.user.username, 
                         tweet.replyCount, tweet.retweetCount, tweet.lang, 
                         tweet.source, tweet.likeCount])
tweets_df1 = pd.DataFrame(tweets_list1, columns=['Datetime', 'Tweet Id', 'URL', 'Text', 'Username', 'ReplyCount',
                                                 'RetweetCount', 'Language', 'Source', 'LikeCount'])

st.button('Show DataFrame')
st.dataframe(tweets_df1.head())

st.cache
def convert_df(df):
    return df.to_csv().encode('utf-8')
csv = convert_df(tweets_df1)
st.download_button(
    label="Download data as CSV",
    data=csv,) 

myclient = pymongo.MongoClient("mongodb://localhost:27017/")

st.button('Upload to MongoDB ?')
data_dict= tweets_df1.to_dict(orient= "records")
mydb = myclient[user_name]
mydb.mycol.insert_many(data_dict)
    
st.write('File uploaded in MongoDataBase as : ',user_name )
