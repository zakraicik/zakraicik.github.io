---
date: 2022-11-02 00:00:00
title: Text Cleaning for NLP Projects
description: Using Python to Prepare Tweets for NLP Tasks
tags:
  - data-preprocessing
  - python
  - nfl-tweets
image: /uploads/jeshoots-com---zmnefoi3k-unsplash.jpg
---
# Cleaning NFL Tweets

*This blog post outlines how we will use clean tweets collected using our [tweet downloader](https://zakraicik.xyz/blog/using-tweepy-to-download-download-tweets-related-to-the-nfl) daily. This is one component of our [NFL tweets](https://zakraicik.xyz/tag/nfl-tweets) project.*

Data cleaning or preprocessing is a vital step to any machine learning process. In a way, it helps standardize our data in such a way that it makes it easier for models to identify trends or uncover insights. Data cleaning varies by machine learning task. For example preparing numeric data for a classification task is much different than preparing strings for sentiment analysis.

In this article, we develop a script to clean the tweets we have been collecting using [G](__notset__)[ithub Actions](https://zakraicik.xyz/blog/github-actions-to-schedule-script). Specifically, we will look at the consider the following tasks.&nbsp;

1. Lowercasing strings&nbsp;
2. Removing extra spaces
3. Removing punctuation
4. Removing Contractions&nbsp;
5. Removing stopwords
6. Lematizing words&nbsp;

The complete script that cleans our NFL tweet database can be found [here](https://github.com/zakraicik/nfl-tweets/blob/main/nfl_tweets/get_tweets/clean_tweets.py).&nbsp;

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## Cleaning a Single Tweet

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

In this section, we will use a single tweet to illustrate how each task works. The tweet we will use is&nbsp;

```
The Giants won't lose to this week because the giants are calling for a victory.
```

### Lowercase Strings

In our example tweet, you'll notice a combination of upper case and lower case letters. The word **giants&nbsp;**appears two times, however the case of each appearance is different. If we were to create word count features- leaving the tweet as is would count these two occurrences separately.&nbsp; However, if we lowercase the string, each occurence would be treated the same and we would end up counting this word twice.&nbsp;

Lowercasing strings for NLP helps in the later stages of the NLP application, specifically when counting words or doing feature creation.&nbsp;

To lowercase a string, we simply call `.lower()` on our string. It will look like this.&nbsp;

```
tweet = tweet.lower()
```

The table below shows the result of lowercasing our example tweet.&nbsp;

| Original | "The Giants won't lose to this week because the giants are calling for a victory.&nbsp;" |
| Updated | "the giants won't lose to this week because the giants are calling for a victory.&nbsp; " |

### Removing Extra Spaces&nbsp;

Our example tweet has an extra space at the end of the string. Extra spaces can create noise in the NLP process, so we remove those.&nbsp;

To remove extra space, we call `.strip()` on the string. It will look like this.&nbsp;

```
tweet = tweet.strip()
```

The table below shows our lowercased tweet before and after removing the extra spaces. Notice the trailing space at the end of the string is removed.&nbsp;

| Lowercased | "the giants won't lose to this week because the giants have a bi-week. " |
| Extra Spaces Removed | "the giants won't lose to this week because the giants are calling for a victory." |

### Removing punctuation

The next step in our cleaning process is to remove punctuation. Removing punctuation, similar to lowercasing words, will help ensure all text is treated equally.&nbsp;

To remove punctuation, we will make use of the Python's built in string library. In the code snippet below, we iterate over every character in the string and remove if it exists in `string.punctuation`.

```
tweet = "".join([char for char in tweet if char not in string.punctuation])
```

The table below shows how the string changes after removing punctuation.&nbsp;

| Extra Spaces Removed | "the giants won't lose to this week because the giants have a bi-week." |
| Punctuation Removed | "the giants wont lose to this week because the giants are calling for a victory" |

### Removing Contractions&nbsp;

Contractions combine two words into a single representation. However, to provide the most value to our NLP task, we would rather represent each of the base words individually. As you probably guessed, this is to help ensure all text is treated equally.&nbsp;

Please note, it would probably be smarter to do this step prior to removing punctuation. However, in our example the effect is the same.&nbsp;

The remove contractions, we will use a package called `contractions`. This package can be easily installed as follows.&nbsp;

```
pip install contractions
```

Once installed, we apply to the string similarly to how we have applied our other preprocessing tasks.&nbsp;

```
tweet = contractions.fix(tweet)
```

The table below shows how the string changes after removing contractions.&nbsp;

| Punctuation Removed | "the giants wont lose to this week because the giants have a biweek" |
| Contractions Removed&nbsp; | "the giants will not lose to this week because the giants are calling for a victory" |

### Removing Stopwords

Stop words are a set of commonly used words in any language. In NLP, stop words are used to eliminate unimportant words, allowing applications to focus on the important words instead.&nbsp;

To remove stop words, we will use a package called [nltk](https://www.nltk.org/). This package is provides a commonly set of nltk tools and methods. This package can be easily installed as follows.&nbsp;

```
pip install nltk
```

After installing, you will need to install the stop words. You can do this by adding the following lines of code to your script.&nbsp;

```
nltk.download("stopwords")
```

We will also need to make sure we import both the stopword listing as well as the NLTK tokenizer. Tokenizaiton is the process of splitting the sentence into individual words (e.g "Tokens"). We will tokenize our tweet, and then compare each token to the list of stop words. We will exclude any word that is in the stopword list.&nbsp;

```
from nltk import word_tokenize
from nltk.corpus import stopwords
stopwords = stopwords.words("english")

cleaned_tweet = []
for word in word_tokenize(tweet):
    if word not in stopwords:
        cleaned_tweet.append(word)

cleaned_tweet = " ".join(cleaned_tweet)
```

The table below shows the tweet before and after removing stopwords.&nbsp;

| Contractions Removed&nbsp; | "the giants will not lose to this week because the giants are calling for a victory" |
| Stopwords Removed&nbsp; | "giants lose week giants calling victory" |

Ouch\! It looks like removing the stop words has completely changed the meaning of this tweet. Although it is common practice in NLP to remove stop words, for our purposes, we will skip this step. We need to ensure the semantics of the tweet is maintained.&nbsp;

### Lemmatizing words&nbsp;

Lemmatization involves reducing tokens to their root word. Like most of our other cleaning tasks, the aim of lemmatization is to standardize our word set and ensure our words are treated equally.&nbsp;

We will again use&nbsp;[nltk](https://www.nltk.org/)&nbsp;for this task. The code below shows how to import, initialize, and work with the nltk lemmatizer. Please note that the lemmatizer requires parts of speech tags to work correctly. I am assuming you have already installed nltk using the instructions from the stopwords section.&nbsp;

```
from nltk import pos_tag, word_tokenize
from nltk.stem import WordNetLemmatizer
lemmatizer = WordNetLemmatizer()
cleaned_tweet = []

for word, tag in pos_tag(word_tokenize(tweet)):

    wntag = tag[0].lower()
    wntag = wntag if wntag in ["a", "r", "n", "v"] else None

    if not wntag:

        lemma = word

    else:

        lemma = lemmatizer.lemmatize(word, wntag)

    cleaned_tweet.append(lemma)

cleaned_tweet = " ".join(cleaned_tweet)
```

The table below shows the tweet before and after lemmatization.&nbsp;

| Contractions Removed | "the giants will not lose to this week because the giants have a biweek" |
| After Lemmatization | "'the giant will not lose to this week because the giant be call for a victory" |

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## Cleaning Multiple Tweets

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

We have walked through a cleaning example for a single tweet, but how do we apply it to a dataframe of tweets? We can use an `apply`&nbsp;function on a dataframe\! I have combined all our preprocessing steps into a single function below.&nbsp;

```
def clean_text(tweet: str) -> str:

    tweet = tweet.lower()

    tweet = tweet.strip()

    tweet = contractions.fix(tweet)

    tweet = "".join([char for char in tweet if char not in punctuation])

    cleaned_tweet = []

    for word, tag in pos_tag(word_tokenize(tweet)):

        wntag = tag[0].lower()
        wntag = wntag if wntag in ["a", "r", "n", "v"] else None

        if not wntag:

            lemma = word

        else:

            lemma = lemmatizer.lemmatize(word, wntag)

        cleaned_tweet.append(lemma)

    cleaned_tweet = " ".join(cleaned_tweet)

    return cleaned_tweet
```

We can simply apply this function to our dataframe of tweets to clean each tweet.&nbsp;

```
df["cleaned_tweet"] = df.apply(lambda x: clean_text(x["tweet"]), axis=1)
```

Again, the complete working example can be found in my Github repo [here](https://github.com/zakraicik/nfl-tweets/blob/main/nfl_tweets/get_tweets/clean_tweets.py).&nbsp;

Feel free to reach out with any questions using the [contact](https://zakraicik.xyz/contact/) page or hitting me up on any of my social links\!