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

- [Exploratory Data Analysis](#heading-2)

- [Feature Engineering](#heading-3)

- [Modeling](#heading-4)

## <a name="heading-1"></a>Overview

I wanted my next project to be something that pertains to current events. With all the news about Russia's focus on United States socail media, I figured that would be a good place to start. I was able to find a dataset released by fivethirtyeight containing 3 million tweets from a company called Internet Research Agency that have since been deleted from Twitter. 

The Internet Research Agency (IRA) is a Russian company located in a single building in Saint Petersburg, Russia. An post on The New York Times found [here](https://www.nytimes.com/2015/06/07/magazine/the-agency.html) sheds some light on what the company did. Their main business was troll farming. The employees were given a quota of tweets that they were required to produce every day. They managed multiple accounts each of which were directed towards different focuses. Their main goal was to create a toxic environment for information outlets. If people doubt the accuracy of anything on the internet, it would reduce the flow of information.

Despite what some may think, IRA was not just sending tweets trying to push a single agenda. They were more focused on positive and negative comments directed to all political parties. My goal with this project is to see if I can uncover structure within the company and view what their network of accounts looks like.