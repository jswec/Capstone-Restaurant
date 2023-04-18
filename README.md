# Yelp Report Cards

Welcome to a project i’m calling the Yelp Report Cards. In this project, it is my aim to produce a business value-add to Yelp’s current business owner services. Using NLTK’s Vader Sentiment Analysis and combining it with Genism’s LDA library, I created a quantitative report card for restaurants to determine areas in which they are doing well, and areas in which they can improve on.

## The Business Case

Yelp is quite possibly the largest review site in the world for local businesses, encompassing all sorts of businesses from doctor’s offices to cleaning services. At its core however, Yelp began as a crowd-sourced review site, democratizing the restaurant review process and bringing greater exposure to restaurants all over the world by soliciting user-generated content for the restaurants in its database.

Over time, the popularity of Yelp as a restaurant information resource has led to the creation of millions of points of data. This has somewhat diluted the impact of Yelp reviews for a restaurant and led to some discontent from restaurant owners. Below are 3 of many points that lead to dissatisfaction with Yelp by business owners.

**1.  Law of Large Numbers**

Due to the sheer volume of reviews, every review that goes into a restaurant ends up stabilizing their rating. There it stays forever, usually at a 4. At this point, most restaurants shoot for a 4, and only the truly best like Le Bernadin (3 Michelin Stars), manage to get a 4.5. Restaurants with a 5 star review typically are newer restaurants with less than 30 reviews, and a lot of chefs believe those mostly come from friends of the restaurant. You can see the distribution in reviews in the below graph. It is assumed 1 and 2 star restaurants usually end up closing, but I can assure you, my local Chinese take out is open and busy as ever. This may also be bias from Yelp’s API feeding me restaurants with a higher review count.

<img src="Images/star_hist_report_card.png">

**2.  Extreme Volatility**

Yelp being the designated ground for people looking to respond in some way or another leads to a lot of praise, or angry passive aggressive content. People who use Yelp for the first time or the thousandth time are usually motivated by extreme experiences, either positive or negative, and as such their reviews tend to reflect those experiences, without taking into account the remainder of the dining experience.

**3. Poor representation**

As described above, due to the focus on a single issue we see a large range of positive and negative reviews that aren't properly reflective of the entire dining experience.  These are not reviews as we have come to commonly expect, noting positives and deficiencies.  As such, many restaurants see a lot of 5 star or 1 star reviews which they feel is unbalanced and affects their total score.

## The Question

Given that we have established some of the issues many restaurants face when it comes to dealing with online reviews, is there a way in which Yelp can add value for these restaurants and provide workable feedback?

Yelp holds millions of user reviews, and using NLP we can take this data and provide comprehensive quantitative metrics for a restaurant without having to spend hours hand sorting reviews. Using this metric, restaurants can hone in on problems and work to improve their business.

## The Process

The process for creating these metrics are quite simple, and outlined in the below graph:
<img src="Images/flowchart.png">

## Step 1:  Gathering Data

Using Yelp's API, I obtained the information and more importantly, the restaurant_id for each restaurant in a given borough.  This provided me with a list of 1000 restaurant_ids that I could then use to scrape 5 pages or at least 100 reviews for restaurants that had that many reviews.  Some restaurants scraped did not have 100 reviews, and other restaurants had more than 100 reviews due to users updating their reviews.  Some of these updated reviews were positive, and some were negative.

Final counts were as follows:

Manhattan: 115,588 Reviews
Queens: 94,301 Reviews
Brooklyn: 100,166 Reviews
Staten Island: 51,490 Reviews
Bronx: 59,150 Reviews

Both Staten Island and the Bronx were the least responsive on Yelp.  I believe this has something to do with my belief (discussed [here](https://towardsdatascience.com/mo-data-mo-money-a1272f653046)) that most people tend not to leave Yelp reviews unless they have a significant dining experience.  These restaurants, being in home territory, are providing daily meals for the residents, and thus they are largely ignored by locals on Yelp.  Dining in Manhattan is widely considered to be an experience, or special occasion and thus is more likely to garner a response on Yelp.  In addition, consulting with a local Staten Island expert, I was informed that possibly due to Staten Island's largely residential nature, most residents either drive across the bridge into Brooklyn for unique dining experiences, or stay home to eat.  This may contribute to Staten Island's low response rate.

## Vader

After gathering the data, the first step I took was to run the data through NLTK’s Vader Sentiment Analysis. Vader is an extremely simple library to use, trained on Twitter Data. As such, it does not require much data preparation as things like punctuation, capitalization and random stop words help it determine sentiment score.

I did not focus on the number of stars given for the review, because I feel the star rating is arbitrary, and there are no guidelines. Its simply a representation of how mad or happy the reviewer is without providing us with a measurable metric to identify specific problems. For example, a 3 star rating is ambiguous, a person could praise the food, but mention 2–3 things that they were unhappy with at the same time.

Vader was done using a simple lambda function that returned to us a set of 4 scores for each review, a positive, neutral, negative and compound score. The distributions were a little lopsided due to the nature of responses one typically sees on Yelp. The below graph is a sampling of the frequency of average compound reviews.

<img src='Images/vadercompoundaveragemanhattan.png'>

As we can see, there are more reviews featuring positive sentiment than negative.  That's ok.  So long as we work within this acknowledged framework, we can comprehend the eventual scores.

For those unfamiliar with Vader, below is a snapshot of some of our restaurant reviews that have been "vaderized":

<img src='Images/vaderyelpsnapshot.png'>

To illustrate how Vader sentiment analysis works, below are sample reviews from the restaurant with the highest average compound score, Fish Cheeks (.945), and the lowest, Di Fara Pizza (.511).

<img src='Images/Screen Shot 2019-10-23 at 9.40.46 PM.png' height='45%' width = '45%'><img src='Images/Screen Shot 2019-10-23 at 9.41.21 PM.png' height='45%' width = '45%'><img src='Images/Screen Shot 2019-10-23 at 9.43.05 PM.png' height='45%' width = '45%'><img src='Images/Screen Shot 2019-10-23 at 9.45.04 PM.png' height='45%' width = '45%'>

As you can see, there are a lot of superlatives associated with the food at Fish Cheeks, people are effusive in their praise. For Di Fara Pizza on the other hand, one of the most famous pizza restaurants in NYC, many people praise the food, but are also dissatisfied with the lines, as the wait is typically 45 minutes to 1.5 hours for a slice of pizza. This also lends weight to my assertion that stars are too ambiguous. There are clearly problems with service at DiFara, but many of the reviews are 5 stars, and in fact, its overall score is 4 stars.  A business owner would see their 4 stars and assume all is well.  Using review stars, thus, would give us a false positive on identifying service problems if I were to use the star system to determine sentiment.

## LDA

The next step in our process is using Latent Dirichlet Allocation.  This is an unsupervised machine learning tool that samples the corpus (all the text) of our data and attempts to derive common topics.  I trained this model on restaurants where service was mentioned, and this comprised only 30% of my overall data.  I instructed the LDA to find 15 topics, since I knew service was underrepresented in my data, so I was looking for at least 2 topics that represented service, and the rest I needed to ensure that the entirety of cuisine on offer in NYC was represented.  It would have been an epic failure if my model was to come across a review mentioning the food and fail at identifying it.

LDA is an exhaustive process that requires constant tuning and rerunning in order to ensure the model is trained sufficiently. In addition to cleaning, lemmatizing, and tokenizing, I also needed to remove a multitude of stop words beyond those found in the standard stopword library. In the end, I needed to add over 1000 additional stop words per borough in order to ensure my model was sufficiently prepared. Words I added to the stop list included all parts of speech that were not relevant to the process like “brother”, “sister”, “steve”, “delicious”, “maybe”, “bomb”, and “scrumptious”. I wanted to ensure food and service oriented words were included, but did make sure to leave off the stop list service related adjectives in order to expand the range of service. My final topic list is below:

<img src='Images/Topics/Manhattantopics.png' height='75%' width = '75%'>

Once the topic list was finalized, I was able to apply the model to the entirety of my reviews and assign a topic number to the highest scoring topic for each review, as seen below:

<img src='Images/LDAtopicsassigned.png'>

## Putting it Together

The final part of the project involved loading the Vader and LDA dataframes and writing a function to tally each one and create a score.  The code is below:

```python
#df is topic modeled restaurant reviews dataframe
#vd is vaderized restaurant reviews dataframe
#maintaining index integrity is important for this function

#establishing food vs service topics
food = [1, 2, 3, 4, 5, 6, 7, 10, 11, 12, 13, 14]
service = [0, 8, 9]
#iterable
nums = [str(s+1) for s in range(139)]


reportcard = []
for row in df[['restaurant_name']].itertuples():
    #these are inside to reset after each restaurant
    numreviews = 0
    badfood = 0
    goodfood = 0
    badservice = 0
    goodservice = 0
    for j in nums:
       #check for Nans
        if not np.isnan(df.iloc[row[0]]['lda_review_'+j]):
            #if integer version of topic number in this cell[0] is in service
            if int(df.iloc[row[0]]['lda_review_'+j]) in service:
                #and if compound score is less than .5, add a point to bad service
                if vd.iloc[row[0]]['compound'+j]<.5:
                    badservice +=1
                else:
                    #otherwise add a point to good service
                    goodservice +=1
            #if integer version of topic number in this cell[0] is in food
            elif int(df.iloc[row[0]]['lda_review_'+j]) in food:
                #and if compound score is less than .5, add a point to bad food
                if vd.iloc[row[0]]['compound'+j]<0.5:
                    badfood +=1
                #otherwise add a point to good food.
                else:
                    goodfood +=1
            else:
                #if all that fails, let me know what failed
                print(int(df.iloc[row[0]]['lda_review_'+j]))
            #track for number of reviews in each row for averaging purposes
            numreviews += 1
    
    #append all this to a dictionary in the following fashion
    reportcard.append({'restaurant': row[1], 
                        'posfood_score': goodfood/numreviews, 
                        'negfood_score': badfood/numreviews,
                        'posservice_score': goodservice/numreviews, 
                        'negservice_score': badservice/numreviews})
```

The final result is stored in a dictionary that we can than convert to a json file and export to use in other applications.  I assembled the score as a percentage of total reviews, such that we can tell restaurants:

"Of the people who left a review, 60% said they loved your food, but 30% said they hated it." 

Further distallation can be done to simplify the score, but I feel this provides a sufficient snap shot for restaurant owners to discern where they are relative to the baseline as time goes on.

All this data was put in a streamlit front end wrapper that allows users to select a restaurant from a dropdown menu and see the scores for each restaurant below:

<img src='Images/frontendsample.png' height='60%' width = '60%'>

You can play with it at the below link:  
https://desolate-ocean-14363.herokuapp.com/

## Conclusions

This project was a fun exercise that combines my passion for food and restaurants in general with my budding data science skills.  There were also some valuable lessons learned.  I now know that LDA is not a model that I want to use in a limited time frame, and could possibly have better sped things up using TFIDF and using a machine learning classifier.  In this case, I used LDA because I wanted to stretch my capabilities and venture into unsupervised learning.  I also enjoyed the easy interpretability of LDA that gave me greater control on fine tuning the model.  In the end, I ended up using TF-IDF in combination with LDA in order to better bubble up service as my topic.

Further expansion of this project will include review snapshots so that restaurant owners can get a sampling of specific events they may want to address, in addition, it may help them better understand the data.  In addition, I'd like to compile an overall report card by borough to determine if each borough has specific deficiencies.  Lastly, I'd like to refine my data a little more and explore if overall rating is affected by these topics.
