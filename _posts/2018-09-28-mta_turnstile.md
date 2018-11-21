---
title: "Optimizing New York City's Subway Traffic"
date: 2018-09-28
tags: [Python]
header:
    overlay_image: "/assets/images/mtaproject/subway.jpg"
excerpt: "Optimizing Citymapper routes using MTA turnstile data"
---
## Table of Contents

- [Overview](#heading)

- [Data Acquisition](#heading-1)

- [EDA](#heading-2)

<!-- toc -->

## Overview

The New York City subway system is a notoriously busy part of everyone's commute. Living in NYC has given me first hand experience of this. The biggest pressure point of commuting for me is getting stuck going in and out of the station. While it can only be for a couple minutes, those minutes drag on forever as I get sandwiched between people going both directions. This issue is the main focus of this post.

For my first project at Metis, my teams goal was to utilize MTA turnstile data to solve a problem of our choosing. We decided to pose as consultants working with the mobile app Citymapper and see if we could use traffic through turnstiles to direct commuters through less used entrances and exits while still being convenient to their trip.

## Data Acquisition

The data that we utilised for this project can be found at the [MTA's website](http://web.mta.info/developers/turnstile.html). The files for their turnstile data are located in multiple different links. To make it easier to grab all the data, I wrote a quick script that used Beautiful Soup to grab the link urls and download the first couple of 2018 to work with.

```python
# get the page with all turnstile links
resp = requests.get('http://web.mta.info/developers/turnstile.html')
soup = BeautifulSoup(resp.text, 'lxml')

# read through html finding link tags
text_urls = []
for link in soup.findAll('a', href=re.compile(r'data/nyct/turnstile/turnstile_')):
    text_urls.append('http://web.mta.info/developers/'+link['href'])

# Only use from 2018
url_2018 = []
for u in text_urls:
    if re.findall(r'turnstile_18', u):
        url_2018.append(u)
i = 1
for url in url_2018[:5]:
    if i == 1:
        df_import = pd.read_csv(str(url))
        print("Downloaded " + url + " from MTA website")
    else:
        df_import = df_import.append(pd.read_csv(str(url)))
        print("Downloaded " + url + " from MTA website")
    i += 1
```

The MTA's data isn't the easiest to understand once you first look at it. The data the MTA collected reflects the condition of their subway system. If you aren't from NYC, it is not great. Overall there are 11 different fields they collect data for. The focused features of this project can be seen below:

![image-center](/assets/images/mtaproject/initial_df.png)

- **C/A**: The **control area/booth name** for the booth at a given station. Each station could have multiple C/A's.
- **Unit**: The **remote unit ID** of a station.
- **SCP**: Subunit/Channel/Position. This is the **specific address for any given turnstile.**
- **Station**: The name that was assigned to a subway station by operations planning.
- **Linename**: A string that contains all the **train lines stopping at that station.**
- **Division**: The line that the station originally belonged to.
- **Date**: The date that the data was audited.
- **Time**: The **time that data was recorded** is in increments of four hours. These times can be staggered to prevent an overload of audit readings to the system.
- **Desc**: Description of the audit event. Can be "REGULAR" for normal times or "RECOVR AUD" if an audit was missed and recovered.
- **Entries**: A **cumulative entry value** for each turnstile. This is measured since the inception of the device. The memory of the turnstile is sometimes deleted or counts backwards from roll over or replacement.
- **Exits**: A **cumulative exit value** for each turnstile. Contains similar issues to the entries.

## EDA
