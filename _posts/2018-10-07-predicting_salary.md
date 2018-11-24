---
title: "Predicting Compensation"
date: 2018-10-07
tags: [Python, Scrapy]
header:
    overlay_image: "/assets/images/glassdoorproject/silverwall.jpg"
excerpt: "Predicting expected salary for Data Science related jobs from Glassdoor postings"
---
## Table of Contents

- [Overview](#heading-1)

- [Data Acquisition](#heading-2)

- [EDA](#heading-3)

- [Calculations/Findings](#heading-4)

- [Conclusions](#heading-5)

- [Limitations](#heading-6)

## <a name="heading-1"></a>Overview

Data Science has become a popular subject in recent years. The title of Data Scientist has been adopted from many companies, however the actual skills needed seem to vary job to job. With my past couple months focused on data, I thought it would be appropriate to gather as much data on data related positions as I could. The information I gather will hopfully tell me what skills are the most important to Data Scientists and what the salary should be for certain sets of skills.
 
## <a name="heading-2"></a>Data Acquisition

Glassdoor has been my main site of choice for job searches, so I figured I would start there. In order to get the best overall idea of skills, I needed to gather a lot of data. I attempted to use the Selenium/Beautiful Soup route initially to scrape the data, but Glassdoor is not fond of you scraping their web pages. Selenium is also somewhat slow for what I needed.

I decided to utilize a very useful web scraping framework called Scrapy. The public github for any Scrapy information can be found [here](https://github.com/scrapy/scrapy). It is a very useful tool and I recommend looking into it if you need to get large amounts of data from webpages. 

Scrapy has a very convenient setting that will cycle IP addresses and delay downloads to get past any IP blockers that are set in place. I set up a quick script to scrape the newest bunch of proxies from a free proxy site, so I wouldn't have to keep going back more than once.

```python
def get_proxies():
    url = 'https://free-proxy-list.net/'
    page = requests.get(url)
    parser = fromstring(page.text)
    proxies = set()
    for i in parser.xpath('//tbody/tr')[:50]:
        if i.xpath('.//td[7][contains(text(),"yes")]'):
            #Grabbing IP and corresponding PORT
            proxy = ":".join(['http', '//'+i.xpath('.//td[1]/text()')[0], i.xpath('.//td[2]/text()')[0]])
            proxies.add(proxy)
    return proxies
```

Another useful feature is the multiple download starts. I quickly found out that glassdoor only shows the first thousand job postings, after that the pages will 404. A thousand job posts is decent, but many of those may not even be related to data positions. Instead of setting the scraper to search the whole country for Data posisions, I gave it six different state searches: NYC, California, Seattle, Colorado, Texas, and Virginia. Scrapy would simultaneously go through each of these pages, so in the end I had around 6,000 job postings to work with.

## <a name="heading-3"></a>EDA