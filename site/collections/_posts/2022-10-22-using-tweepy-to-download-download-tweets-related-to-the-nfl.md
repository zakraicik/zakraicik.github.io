---
date: 2022-10-22 00:00:00
title: Getting a Sample of NFL related Tweets
description: 'Using Tweepy to download tweets related to the NFL, good or bad. '
tags:
  - get-data
  - nfl-tweets
  - tweepy
image: /uploads/nfl-twitter.jpeg
---
# Downloading NFL Tweets&nbsp;

*This blog post outlines how we will use [tweepy](https://www.tweepy.org/)&nbsp;to develop a dataset for our [NFL tweets](https://zakraicik.xyz/tag/nfl-tweets) project. This is just one component of a larger project*

## Installing Tweepy&nbsp;

---

Tweepy is an easy-to-use Python library for accessing the Twitter API. The API class provides access to the entire twitter RESTful API methods. Each method can accept various parameters and return responses. Using the API, you can programmatically access things like tweets, spaces, and lists.&nbsp;

To install tweepy, you can use the following command `pip install tweepy`

More information about other ways to install tweepy can be found [here](https://docs.tweepy.org/en/latest/install.html)&nbsp;.

## Setting up Twitter API Access

---

The Twitter API can be used to programmatically retrieve and analyze Twitter data. In order to set be able to call the API within scripts we write, the firs thing we need to do is set up a twitter to developer project.&nbsp;

### Setting up a Twitter Developer Project

To do so, follow these steps:

1. In order to get started with the new Twitter API, you need a developer account. If you do not have one yet, you can&nbsp;[sign up for one](https://developer.twitter.com/en/portal/petition/essential/basic-info).
2. Next, in the&nbsp;[developer portal](https://developer.twitter.com/en/portal/dashboard), create a new Project. Once the project is created, you will need to connect it to an&nbsp;[App](https://developer.twitter.com/en/docs/apps/overview). AN App is just container for your API Keys that you need in order to make an HTTP request to the Twitter API.

Once you have created a developer project, your developer dashboard should look something like this.&nbsp;

![](/uploads/screen-shot-2022-10-24-at-6-54-44-am.png){: width="1173" height="622"}

### API Access Levels

The API has varying levels of access, where the level of access a user has dictates the amount they can use the API and the types of information they can access.&nbsp;

At the time of writing this article, there are 3 levels of access for the API. I highlight some of key differences below.&nbsp;

* **Essential**
  * Developer accounts come standard with this level of access
  * Retrieve up to 500k Tweets per month
  * Only supports app-only and user context authentication methods
* **Elevated:&nbsp;**
  * Retrieve up to 2 million Tweets per month
  * Supports all authentication methods&nbsp;
* **Academic Research:**
  * Retrieve up to 10 million Tweets per month
  * Access to full tweet archive & advanced filter operators
  * Supports all authentication methods&nbsp;

For this project, we will request developer access. This will enable us to collect more tweets for our training data. Academic research would be nice, but it is hard to get. API users can apply for elevated access within the [twitter developer portal](https://developer.twitter.com/en/portal/dashboard).&nbsp;

## Using Tweepy to Download Tweets&nbsp;

---

At this point, we are ready to use tweepy to download tweet extracts for our project. We have the correct access and have our project setup. The completed extraction script can be found [here](https://github.com/zakraicik/nfl-tweets), but we will walk through each step in greater detail below. As a disclaimer, this script is not written in such a way that it can easily handle errors. You can modify this script to have better error handling if you wish.&nbsp;

### Required Imports

For this script, we will import these required packages&nbsp;

```
import tweepy as tw
import pandas as pd
import logging
import os
import datetime
```

### Tweepy Authentication

Next, we want need to point our script to the app we set up in the previous steps. To do this, we will use app only authentication. You can find your&nbsp;`BEARER_TOKEN`&nbsp;within the twitter developer portal. I stored this token within my environment variables so it can easily interact with Github actions in the next step of this project. If you have never worked with environment variables before, this [article](https://towardsdatascience.com/the-quick-guide-to-using-environment-variables-in-python-d4ec9291619e)&nbsp;provides a helpful overview of how to use environment variables within Python scripts.&nbsp;

If your&nbsp;`BEARER_TOKEN`&nbsp;is stored within your environment variables, you can access it within your script in the following way.&nbsp; Additionally, we set up some basic logging here so we can see what the script is doing as it runs.&nbsp;

```
BEARER_TOKEN = os.getenv("BEARER_TOKEN")

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)
```

Now, we need to initialize our Tweepy client. This client will authenticate our script using the&nbsp;`BEARER_TOKEN`&nbsp;. We will do this via a function titled&nbsp;`initalize_client`. You can also see we use the logger to confirm the client was set up correctly. This function returns an authenticated tweepy client that will enable our script to interact with the API.&nbsp;

```
def initalize_client() -> tw.Client:

    client = tw.Client(bearer_token=BEARER_TOKEN)

    logger.info(" Client authenticated and initialized.")

    return client
```

### Searching Twitter

Now that our client has been set up, we will pass it to a function titled search. The signature for this function is&nbsp;`def search(client: tw.Client, query: str, max_results: int, limit: int)`. We will briefly describe each of these inputs below.&nbsp;

* **client**\: Authenticated twitter client we will use to execute our query. This is created in the&nbsp;`initalize_client`​​​ function.
* **query**\: The twitter search query. Using elevated access, we have some the ability to use some basic&nbsp;[operators](https://developer.twitter.com/en/docs/twitter-api/tweets/search/integrate/build-a-query). These operators can be used to filter the results of our twitter query.&nbsp;
* **max\_results**\: Maximum number of tweets to return per page&nbsp;
* **limit**\: Total number of tweets to return (number of pages to query = total number of tweets divided by max results per page).&nbsp;

There are multiple piece of our `search` function.

* Paginator
* Tweet filter&nbsp;
* Writing to dataframe

We will look at each of these components in more detail.&nbsp;

#### Paginator

We will use the paginator to search recent tweets using our search criteria. Specifically, the lines of code will look like this&nbsp;

```
tweets = tw.Paginator(
    client.search_recent_tweets,
    query,
    tweet_fields=["context_annotations", "created_at"],
    max_results=max_results,
).flatten(limit=limit)
```

We tell our `client` to hit the&nbsp;[search recent tweets](https://developer.twitter.com/en/docs/twitter-api/tweets/search/api-reference/get-tweets-search-recent)&nbsp;endpoints. We will pass a query to the endpoint, specifying what we want to search for. We will limit our search to `max_results`&nbsp;per page, and collect `limit` tweets in total. Additionally, we will pull in some additional information available for [tweet objects](https://developer.twitter.com/en/docs/twitter-api/v1/data-dictionary/object-model/tweet)&nbsp;(`created_at`, `context_annotations`).&nbsp;

For this project, the query we pass to the paginator is&nbsp;`#nfl -is:reply -is:retweet lang:en -has:media`. This could probably be improved if you really wanted to spend time here. We use some basic operators to ensure our tweets are in english and exclude media (GIFs, videos, etc).

The paginator will return generator of [tweet objects](https://developer.twitter.com/en/docs/twitter-api/v1/data-dictionary/object-model/tweet).&nbsp;

#### Tweet Filtering

Tweet objects come with some build in [context annotations](https://developer.twitter.com/en/docs/twitter-api/annotations/overview)&nbsp;we can use to eliminate tweets from our query that seem unrelated. In our example, since we are looking for tweets related to the NFL, we limit our results to relevant annotations.&nbsp;

We iterate over each&nbsp;[tweet object](https://developer.twitter.com/en/docs/twitter-api/v1/data-dictionary/object-model/tweet)&nbsp;in our paginator result, and append any relevant tweet to a list so that we can store our results in a dataframe.&nbsp;

```
relevant_tweets = []
created_at = []

for tweet in tweets:

    if len(tweet.context_annotations) <= 0:

        pass

    else:

        for context_annotation in tweet.context_annotations:

            if "entity" in context_annotation:

                if context_annotation["entity"]["name"].lower() in [
                    "nfl",
                    "nfl football",
                    "gambling",
                    "american football",
                    "Sports betting",
                ]:

                    relevant_tweets = relevant_tweets + [tweet.text]
                    created_at = created_at + [tweet.created_at]
```

#### Storing Results in Pandas Dataframe

To make things easy, we will have our `search` function return a pandas dataframe containing the tweet and the date the tweet was created.&nbsp;

```
df = pd.DataFrame(relevant_tweets)

df.columns = ["tweet"]

df["created_at"] = created_at

df.drop_duplicates(inplace=True)
```

#### A Complete Search Function

Putting it all together, our search function now looks like this&nbsp;

```
def search(client: tw.Client, query: str, max_results: int, limit: int) -> pd.DataFrame:

    tweets = tw.Paginator(
        client.search_recent_tweets,
        query,
        tweet_fields=["context_annotations", "created_at"],
        max_results=max_results,
    ).flatten(limit=limit)

    logger.info(" Tweets returned.")

    relevant_tweets = []
    created_at = []

    for tweet in tweets:

        if len(tweet.context_annotations) <= 0:

            pass

        else:

            for context_annotation in tweet.context_annotations:

                if "entity" in context_annotation:

                    if context_annotation["entity"]["name"].lower() in [
                        "nfl",
                        "nfl football",
                        "gambling",
                        "american football",
                        "Sports betting",
                    ]:

                        relevant_tweets = relevant_tweets + [tweet.text]
                        created_at = created_at + [tweet.created_at]

    logger.info(" Relevant tweets extracted")

    df = pd.DataFrame(relevant_tweets)

    df.columns = ["tweet"]

    df["created_at"] = created_at

    df.drop_duplicates(inplace=True)

    logger.info(" Sent relevant tweets to DF.")

    return df
```

The resulting dataframe will look something like this.&nbsp;

![](/uploads/screen-shot-2022-10-24-at-8-15-42-am.png){: width="545" height="156"}

### Executing the Script

---

Finally, we combine all of the functions detailed above to run our script.&nbsp;

* Initialize client establishes our connection to the twitter API
* Search returns a dataframe of tweets of interest&nbsp;
* We write the dataframe to a parquet file in the data folder of this repo. As I will write in and future blog post, we will use Github actions to run this script daily and create a new parquet file for each day. With that in mind, that is why we name the file as `tweets_<date>.parquet`

```
if __name__ == "__main__":

    working_dir = os.getcwd()

    data_dir = os.path.join(working_dir, "nfl_tweets/data/")

    logger.info(" Starting script.")

    client = initalize_client()

    relevant_tweets = search(
        client,
        "#nfl -is:reply -is:retweet lang:en -has:media",
        max_results=100,
        limit=3000,
    )

    file_name = "tweets_" + str(datetime.date.today()).replace("-", "_") + ".parquet"

    if os.path.exists(data_dir):

        relevant_tweets.to_parquet(os.path.join(data_dir, file_name))

        logger.info(
            f" Wrote {relevant_tweets.shape[0]} rows and {relevant_tweets.shape[1]} columns to {file_name}"
        )

    else:

        os.mkdir(data_dir)

        logger.info(f" Created {data_dir}")

        relevant_tweets.to_parquet(os.path.join(data_dir, file_name))

        logger.info(
            f" Wrote {relevant_tweets.shape[0]} rows and {relevant_tweets.shape[1]} columns to {file_name}"
        )
```

Feel free to reach out with any questions using the&nbsp;[contact](https://zakraicik.xyz/contact/)&nbsp;page or hitting me up on any of my social links\!

&nbsp;