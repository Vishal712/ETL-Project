# ETL-Project UCI Data Analytics
### Vishal Patel, Anh Le, Troy Rippeto

## Project Report

### YouTube Trending Videos
Our project used ETL in order to, first, extract data on Kaggle that contains trending YouTube videos in different countries. These videos were specifically featured on the Trending page of the site. We also extracted a table of the top 500 YouTube channels based on SocialBlade. Our goal was to create two collections, one that contained YouTube videos that were on the trending page for all the countries included, and one that uses table of the top 500 channels to determine which of these channels are most successful in order to gauge how influential these videos and channels are.


### Note* Notebook contains all the extraction, transformation, and loading. Please download notebook to view code if the Github preview distorts tables
### Extract:
There were two sources of data, one containing Youtube videos featured on the trending page in different countried, and the other was the top 500 YouTube channels on the site.In order to extract the data for each country, we used a dataset found on Kaggle. The dataset, found here: https://www.kaggle.com/datasnaek/youtube-new, contains different csv files based on different countries. For example, CAvideos.csv contains the YouTube videos that trended only in Canada. For our project we included 3 countries: the US, Canada, and the UK. We chose these 3 countries because they were all primarily English speaking, and thus would share the most. We felt this would also help potential employers and influencers to see how well certain content translates to different areas. 
The other dataset contains the top 500 YouTube channels, found here: https://www.kaggle.com/balaka18/youtubers-popularity-dataset. We used the top_500.csv because we thought it would give a good idea of which channels and videos have a bigger impact or longevity rather than videos that go trending for only a moment. 
We transformed each csv file into a Pandas dataframe, using read_csv(), and started to transform each.

### Transform:

For the first collection, after reading our csv files, we needed to determine which columns we did not need for our videos table for each country. Ultimately, we chose to get rid of the trending date, comments disabled, ratings disabled, video error or removed, and the thumbnail link. We felt that the final collection of videos should be more focused on what the video was rather than what happened to the video, such as the channel disabling comments or likes and dislikes. So the columns that were left were the video id, title, channel title, category number, publish time, tags, views, likes, dislikes, comment counts, and descriptions. 

The next thing we did was replace the column of category id to an actual category name. For example, instead of have a 10, the category column would contain "Music". This would make it easier for someone looking at the collection to tell what type of video had trended. 
In order to do so, we looked up a list of numbers for categories and what they represented, and came across it here on Github: https://gist.github.com/dgp/1b24bf2961521bd75d6c#gistcomment-2714395. We stored this into a text file, and stored it as a dictionary, to store the category id as a key, and the category in English as the value. 
Now we used the .replace() function on each of the 3 tables of videos to replace the category id with the name instead of the number.

Now, using Pandas, we merged all 3 dataframes, originally with the video id as the common column. However, when we viewed the resulting dataframe, it contained many more videos than we had expected, because some videos were listed more than once. This was the biggest challenge we faced during the project, which was determining why the videos that appeared on the trending page for all 3 countries, listed more than once in the aggregated list.
The first thing we noticed was that the video id was not completely the same in each country, so even if we joined on the id, we were getting different videos with the same id. So when we switched the join on title instead, we were able to get a list of unique titles that were shared in all 3 countries. 
The second thing we had to do was drop the likes, dislikes comments and description. The problem with including these was that because the csv files contained videos listed multiple times, as they were on the trending page, they ended up in the merged collection multiple times, with different likes and dislikes, and different amounts of comments. This means we had to remove these categories to reduce repitition in the final collection. As for the description, different countries naturally had different description to each video, so removing this removed repeating videos as well.
Finally, after a drop_duplicates(), we got a Pandas dataframe that contained the title, channel title, category, and tags of each video that trended in the US, Canada and the UK. We ended up renaming these columns as well to remove underscored and special characters. 

For the second collection, containing the top 500 ranked YouTube channels, we first transformed the csv file into a dataframe. Then we merged the dataframe we made earlier with this one, combining on the channel name. Then we removed categories that were specifically about the video, and kept ones that were focued on the channels. For example, we removed the video title, tags, and upload numbers. What we kept instead focused on the channel, as this collection is about determining if the channel is one worth looking into as a sustainable, successful one. We kept the rank of the channel, the grade given, the channel's name, the subscriber count, channel views, and the category for the videos they make. 
The last thing we needed to do was translate the rank from a string to an integer, so that someone using this table could sort it easily based on rank. So we turned the dataframe into a dictionary, and translated each value of rank from a string to an integer, using the split() method, and converting the dictionary back to a dataframe. This means we go from "19th" to 19, for example. We now have two dataframes ready to load.


### Load:
To load the database, we used MongoDB. The reason we chose MongoDB was because it felt more appropriate for this type of data considering its a list of videos with different countries that can all change very quickly. In a month, for example, there could be a trending type of video that many countries gravitate towards, and having the combined trending videos in mongo allows this to updated easily. This also makes it easier to add more countries in the future. As stated earlier, there were many countries in the Dataset we did not include, but we could easily do so by modifying the Python code. Also, because the final collections doesn't have a large number of columns, it makes it easy to query these collections without SQL.

So to load these collections, we used PyMongo to establish a connection to the local host. We then made a client, and connected that client to a new database called ETLYouTube_db. Then we dropped two collections, in case this was being ran again and to avoid adding data again to the same database. Then we converted our first dataframe, with the combined 3 countries' trending videos, to a dictionary using .to_dict('records'). Then using insert_many from Pymongo, we inserted the dictionary to a collection called SharedTrendingVideos. We repeated the process with the other collection of trending videos whose channels were in the top 500, and inserted that into a collection called SharedChannelsTop500.
