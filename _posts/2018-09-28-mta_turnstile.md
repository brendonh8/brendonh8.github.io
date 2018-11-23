---
title: "Optimizing New York City's Subway Traffic"
date: 2018-09-28
tags: [Python]
header:
    overlay_image: "/assets/images/mtaproject/subway.jpg"
excerpt: "Optimizing Citymapper routes using MTA turnstile data"
---
## Table of Contents

- [Overview](#heading-1)

- [Data Acquisition](#heading-2)

- [EDA](#heading-3)

- [Calculations/Findings](#heading-4)

## <a name="heading-1"></a>Overview

The New York City subway system is a notoriously busy part of everyone's commute. Living in NYC has given me first hand experience of this. The biggest pressure point of commuting for me is getting stuck going in and out of the station. While it can only be for a couple minutes, those minutes drag on forever as I get sandwiched between people going both directions. This issue is the main focus of this post.

For my first project at Metis, my teams goal was to utilize MTA turnstile data to solve a problem of our choosing. We decided to pose as consultants working with the mobile app Citymapper and see if we could use traffic through turnstiles to direct commuters through less used entrances and exits while still being convenient to their trip.

## <a name="heading-2"></a>Data Acquisition

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

## <a name="heading-3"></a>EDA

After filtering out and organizing the many glitches that occur in the entry and exit counts, I had the entry and exit counts for each day that I could plot to get an idea of trends in the data. The daily entries show clear trends of weekday and weekend trips.

![image-center](/assets/images/mtaproject/times_sq_entries.png)

Times Square is one of the top visited stations in NYC. The trends each week make sense showing the flow of traffic on weekdays and weekends. The most useful times that would see the largest benefit of optimizing subway traffic would be during the week.

![image-center](/assets/images/mtaproject/days_of_week.png)

## <a name="heading-4"></a>Calculations/Findings

Looking at each control area would give us an idea of the traffic going through specific subway entrances. In order to determine which entrance would be best, we needed to create a metric that would allow us to rate how busy the control areas in each station were. 

![image-center](http://latex.codecogs.com/gif.latex?Rating%3D10000*%5Cfrac%7B%5Cfrac%7BEntries%7D%7BExits%7D%5E2%7D%7BEntries%7D)

This formula we made will weigh stations with more entrances than exits as a lower rating. A better station to enter would be one that has less traffic coming out of it since it is easier to move with the flow of people than against them. This formula proved useful in uncovering the best stations to enter. The plots below show the distribution of entries to exits in Grand Central as well as how our formula rated each of the entrances.

![image-left](/assets/images/mtaproject/distribution.png)![image-right](/assets/images/mtaproject/grand_rating.png)