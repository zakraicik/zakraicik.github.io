---
date: 2022-10-28 00:00:00
title: Github Actions to Schedule Script
description: 'Scheduling the Tweepy script to download NFL tweets using Github actions. '
tags:
  - github
  - nfl-tweets
  - python
image: /uploads/github-actions.png
---
# Downloading NFL Tweets

*This blog post outlines how we will use [Github actions](https://github.com/features/actions) to run our [tweet downloader](https://zakraicik.xyz/blog/using-tweepy-to-download-download-tweets-related-to-the-nfl) daily. This is one component of our [NFL tweets](https://zakraicik.xyz/tag/nfl-tweets) project.*

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## Github Actions

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform built into Github. It can be used for a lot of things including but not limited to

* Building, testing and deploying code
* Automating repetitive tasks
* Managing infrastructure or other third party tools (e.g. [snyk](https://snyk.io/))

Github actions can run on a schedule or as a result of certain events (push, pull request, etc). All Github users have access to Github actions for free until a certain point.

There are many other CI/CD tools available. However, the benefit of Github actions is that the tool is built into Github. You don't have to worry about getting any other tools set up and integrated into your repository.

All Github actions are managed through a YAML file. Each YAML file contains a few key elements, which are lightly described below.

* **Events:** The event specifies what will trigger the workflow. Example events include pushes to a certain branch, pull requests, and more. A complete listing of possible events can be found [here](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows).
* **Jobs:** Jobs represent the steps within a workflow. This can include things like checking out the repo, installing dependencies, etc. All steps run in parallel on the same runner (virtual machine).
* **Steps:** Steps are the individual actions that are run as part of a single job. As mentioned above, these can include things like checking out a repo, install dependencies, etc. Steps can include custom code or running pre-written Github actions.
* **Actions:** This refers to the command to execute on the runner at a given step.
* **Runners:** A runner is the Github Actions server. Servers can be self hosted or hosted by Github.

The combination of these 5 concepts can be used to create a complete YAML file. When writing YAML files, I find this [website](https://codebeautify.org/yaml-validator) helpful to ensure I have structured the file correctly.

A complete Github Actions tutorial can be found [here](https://lab.github.com/githubtraining/github-actions:-hello-world/?scid=7013o000002CceTAAS).

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## Github Actions, In Action

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

Earlier in our [NFL tweets](https://zakraicik.xyz/project/using-tweets-to-make-nfl-bets) project, we wrote a script that searches twitter and downloads a subset of relevant NFL tweets. However, this script only collects a couple thousand tweets when executed locally. I could manually run this script every day to build a database of tweets, however that sounds relatively tedious and annoying.

Instead, I will use Github Actions to automate this data collection effort and automatically execute this script each day. This will grow my dataset by 3,000 tweets each day.

##### Setting up the Workflow Directory

The first step to setting up a Github action is to create a `.github/workflows/` directory in the root of your repository. The workflow directory will house our YAML file.

![](/uploads/screen-shot-2022-11-02-at-7-31-39-am.png){: width="906" height="690"}

##### Writing the YAML File

The first thing we will specify in our YAML file is the `on` argument. This tells Github when we want to execute the steps contained in the YAML file. For this project, we will include two events.

* **Push to Main:** It's not a great practice to push directly to main, but since I'm building this repository myself I am ignoring that concern. Any time I push new code to main this script will execute. We might change this later once we start using development branches to advance other parts of the project.
* **At 11AM UTC:** This action will execute every day at the same time. Since this action is designed to download new tweets every day, this makes sense.

A complete `on` argument like this.

```
name: get-new_data

on:
  push:
    branches:
      - main
  schedule:
    - cron: '0 11 * * *'
```

Next, we will specify the `jobs` argument. Keep in mind, a single YAML file can have multiple jobs. All steps within a job will run on the same VM in parallel. For our purposes, we only want this action to execute our tweets downloader so we only have one job. We name our job download\_tweets and tell Github to run it on the latest version of Ubuntu.

```
jobs:
  download_tweets:
    runs-on: ubuntu-latest
```

Now that we have set up our job, we can specify the steps contained within that job. For our script, we will include the following steps.

1\. **Checkout the Repo:** This checkouts the repository so the action can access it.

```
- name: Checkout Repo
  uses: actions/checkout@v3
```

2\. **Install [Poetry](https://python-poetry.org/) (package manager):** The repository I am building uses poetry to install and manage package dependencies. Therefore, we install it on the GIthub runner.

```
- name: Install Poetry
  run: pipx install poetry
```

3\. **Setup python and Install Our Dependencies:** We will use a predefined action `setup-python@v4` to install poetry and install the dependencies from our `poetry.lock` file. It's worth noting that using this action caches dependencies so they don't need to be installed each time. This significantly speeds up the time it takes to execute the YAML file, which reduces the action minutes logged to github. This ensures we don't have to pay to use the actions.

```
- name: Setup Python & Install dependencies
  uses: actions/setup-python@v4
  with:
    python-version: '3.9.14'
    cache: 'poetry'
- run: poetry install
```

4\. **Execute Our `get_tweets.py` File:** Runs the pythons script that pulls 3,000 new NFL tweets. These tweets are stored in a Pandas DataFrame and eventually written to a parquet file using the naming convention `tweets_<date>.parquet`. We will create a new parquet file each day the action executes.

```
{% raw %}- name: Run get_tweets.py
  env:
    BEARER_TOKEN: ${{ secrets.BEARER_TOKEN }}
  run:
    poetry run python nfl_tweets/get_tweets/get_tweets.py{% endraw %}
```

5\. **Commit and Push:** The updated CSV file only exists locally on the runner. We commit these changes and push them to the remote repository.

```
{% raw %}- name: Commit and push
  env:
      github_token: ${{ secrets.GITHUB_TOKEN }}
  run: |
    git config --local user.email "action@github.com"
    git config --local user.name "GitHub Action"
    git add -A
    git commit -m "Data updated" -a
    git push{% endraw %}
```

The completed YAML file looks like this.

```
{% raw %}name: get-new_data

on:
  push:
    branches:
      - main
  schedule:
    - cron: '0 11 * * *'

jobs:
  download_tweets:
    runs-on: ubuntu-latest
    steps:

      - name: Checkout Repo
        uses: actions/checkout@v3

      - name: Install Poetry
        run: pipx install poetry

      - name: Setup Python & Install dependencies
        uses: actions/setup-python@v4
        with:
          python-version: '3.9.14'
          cache: 'poetry'
      - run: poetry install

      - name: Run get_tweets.py
        env:
          BEARER_TOKEN: ${{ secrets.BEARER_TOKEN }}
        run:
          poetry run python nfl_tweets/get_tweets/get_tweets.py

      - name: Commit and push
        env:
          github_token: ${{ secrets.GITHUB_TOKEN }}
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add -A
          git commit -m "Data updated" -a
          git push
{% endraw %}
```

##### Github Secrets

You might notice a few of the steps above use a piece of code that looks like `${{ secrets.&amp;amp;amp;amp;lt;secret name&amp;amp;amp;amp;gt; }}`. If you remember from the previous [post](https://zakraicik.xyz/blog/using-tweepy-to-download-download-tweets-related-to-the-nfl) detailing our `get_tweets.py` file, we use environment variables to pass API keys to our script and authenticate our client. However, how would Github know these key values if the runner doesn't have access to our local environment variables?

Secrets are encrypted environment variables that you create in an organization, repository, or repository environment. In other words, they are values that can be accessed by the VM as environment variables.

For this project, I stored the Twitter `BEARER_TOKEN` as a secret that can be accessed by the Github action.

![](/uploads/screen-shot-2022-11-02-at-7-32-52-am.png){: width="932" height="338"}

In our `get_tweets.py` script, Github uses this secret to create the `BEARER_TOKEN` variable required by the rest of the script.

<div><div><code>BEARER_TOKEN = os.getenv("BEARER_TOKEN")</code></div><div>&nbsp;</div></div>

Creating a secret is easy. Simply navigate to the settings within your Github repository and find the secrets menu option. Create a new actions secret and you can pass it to your action.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

## Reviewing the Results

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

Once you have created your action, you can view the results of that action. On the home page of your repository, navigate to the actions section.

![](/uploads/screen-shot-2022-11-02-at-7-33-10-am.png){: width="1261" height="453"}

From there, you can see a list of all actions executed within your repository.

![](/uploads/screen-shot-2022-11-02-at-7-33-35-am.png){: width="1836" height="753"}

Clicking on any of these actions will bring up a detailed view of what happened when that action was executed. You'll notice that each step we defined in our YAML file has it's own item and can be expanded to get more detail. For the screenshot below, I expanded the result of the `get_tweets.py` step so we can see how many tweets were added to the extract.

![](/uploads/screen-shot-2022-11-02-at-7-34-02-am.png){: width="1117" height="788"}

Feel free to reach out with any questions using the [contact](https://zakraicik.xyz/contact/) page or hitting me up on any of my social links\!