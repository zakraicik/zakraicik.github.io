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

Tweepy is an easy-to-use Python library for accessing the Twitter API. The API class provides access to the entire twitter RESTful API methods. Each method can accept various parameters and return responses. Using the API, you can programmatically access things like tweets, spaces, and lists.&nbsp;

To install tweepy, you can use the following command \`pip install tweepy\`

More information about other ways to install tweepy can be found [here](https://docs.tweepy.org/en/latest/install.html)&nbsp;.

## Adjusting Tweepy Access&nbsp;

The Twitter API can be used to programmatically retrieve and analyze Twitter data. The API has varying levels of access, where the level of access a user has dictates the amount they can use the API and the types of information they can access.&nbsp;

At the time of writing this article, there are 3 levels of access for the API. I highlight some of key differences below.&nbsp;

* **Essential**
  * Retrieve up to 500k Tweets per month
  * Only supports app-only and user context authentication methods
* **Elevated:&nbsp;**
  * Retrieve up to 2 million Tweets per month
  * Supports all authentication methods&nbsp;
* **Academic Research:**
  * **​​​​​​​**Retrieve up to 10 million Tweets per month
  * Access to full tweet archive & advanced filter operators
  * Supports all authentication methods&nbsp;

For this project, we will elevated access primarily to be able to download more tweets per month. Academic research would be nice, but it is hard to get.&nbsp;

API users can apply for elevated access within the [twitter developer portal](https://developer.twitter.com/en/portal/dashboard).&nbsp;

## Using Tweepy to Download Tweepy&nbsp;

&lt;Writing Later&gt;

&nbsp;

&nbsp;

&nbsp;