---
title: "Uncovering Structure Within a Social Media Troll Company"
date: 2018-11-15
tags: [Python, AWS, NLTK, Spacy, sklearn, Gensim, plotly, NetworkX]
htmlwidgets: TRUE
header:
    overlay_image: "/assets/images/trolltweets/twitter.jpg"
excerpt: "Uncover the topics that the Russian troll company, Internet Research Agency (IRA), drew their attention to on Twitter"
---
## Table of Contents

- [Overview](#heading-1)

- [Data Preparation](#heading-2)

- [Topic Modeling](#heading-3)

- [Modeling](#heading-4)

## <a name="heading-1"></a>Overview

I wanted my next project to be something that pertains to current events. With all the news about Russia's focus on United States socail media, I figured that would be a good place to start. I was able to find a dataset released by fivethirtyeight containing 3 million tweets from a company called Internet Research Agency that have since been deleted from Twitter. 

The Internet Research Agency (IRA) is a Russian company located in a single building in Saint Petersburg, Russia. An post on The New York Times found [here](https://www.nytimes.com/2015/06/07/magazine/the-agency.html) sheds some light on what the company did. Their main business was troll farming. The employees were given a quota of tweets that they were required to produce every day. They managed multiple accounts each of which were directed towards different focuses. Their main goal was to create a toxic environment for information outlets. If people doubt the accuracy of anything on the internet, it would reduce the flow of information.

Despite what some may think, IRA was not just sending tweets trying to push a single agenda. They were more focused on positive and negative comments directed to all political parties. My goal with this project is to see if I can uncover structure within the company and view what their network of accounts looks like.

## <a name="heading-2"></a>Data Preparation

Tweets contain many random spelling errors and characters that need to be cleaned to make analysis easier. There are many Russian tweets in this dataset so in order to extract information I needed to keep only the Enlgish tweets. Once I also removed any emoji's and punctuation, I could start to prepare the data for processing.

Natural Language Processing works much better with generalized words. A common way to do this would be stemming, although it is also the simplest. The idea behind stemming is to derive the words to a base form. However the process of stemming is simple and only chops off the ending of words to try and achieve this goal. I attempted to use NLTK's stemmer and once I extracted word topics, they were incoherent. I decided to use another language generalizer called lemmatization. Lemmatization takes part of speech into consideration so words get reverted to a general form while still maintaining their structure. The idea of stemming and lemmatizing can be seen below:

![image-center](/assets/images/trolltweets/stemlem.jpg)

NLTK has a fairly impressive lemmatizer however for the amount of documents (tweets) I am using, it was performing pretty slow. I decided to use a newer framework called SpaCy that lemmatizes in a different way. NLTK attempts to split the text into sentences, SpaCy constructs a syntactic tree for each sentence which allows for much more information about the words to be retained. An example of this tree can be seen [here](https://explosion.ai/demos/displacy). This made it very easy to remove things like pronouns. 

The final step before converting to a bag-of-words format for NLP is creating N-grams. I decided to only use bi-grams since tweets are generally short sentences and anything higher may not result in being useful. Bi-grams double up words so something like "new york" is captured instead of just reading "new" and "york".

## <a name="heading-3"></a>Topic Modeling

