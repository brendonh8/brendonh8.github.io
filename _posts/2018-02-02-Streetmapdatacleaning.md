---
title: "OpenStreetMap Data Cleaning"
date: 2018-02-02
tags: [data analysis, python, SQL]
header:
    overlay_image: "/assets/images/birds-cold-conifer-917494 (1).jpg"
excerpt: "Cleaning and analysis of user contributed XML map data of Denver, CO."
---

### Map Area: Denver, CO

 - https://mapzen.com/data/metro-extracts/metro/denver-boulder_colorado/
 - https://www.openstreetmap.org/relation/253750

I have always considered moving to Denver, so I decided to use the area for this project. I am interested in the information I will get from the data.

## Problems Encountered in the Map

Outputting the values for various data types revealed many problems that the data had. Some of the problems was formatting, and some was for incorrect entries. I decided to focus on a couple of these problems.

 - Abbreviated street types ("Apache Dr")
 
 
 - Incorrect city names ("Centenn")
 
 
 - Cities with the state included ("Aurora, CO")
 
 
 - Cities in all caps or all lowercase
 
 
 - Invalid phone number with less than 10 numbers
 
 
 - Different phone number formats
 
 
 - Two numbers in the same entry ("303-759-0316;720-666-3971")
 
 

### Cleaning Street Names

Running a function in Python to return a set of all the street names revealed that quite a few street types have been abbreviated. Using regular expressions to find the street names made it easy to find the ending street type of each name. Then all of the abbreviated names were substituted with the full street type name from a list of mapping values 


```python
def update_name(name, mapping):

    m = street_type_re.search(name)
    other_st = []
    if m:
        street_type = m.group()
        if street_type in mapping.keys():
            name = re.sub(street_type_re, mapping[street_type], name)
        else:
            other_st.append(street_type)
    return name
```

### Cleaning City Names

Fixing the city name data used a similar solution as the street names. Using a regular expression to find every instance that CO is included, I was able to substitute it for an empty string. Some of the values were input as unicode so I had to encode those first. Then I could capitalize all the values and apply another mapping function to correct any incorectly entered names


```python
def updateCity(city, cityMapping):
    if type(city) == unicode:
        city = city.encode('ascii', 'ignore')
    # Remove the state at end
    city = re.sub(r'(?i),? (co)*\d*$', '', city)
    city = string.capwords(city)
    city = mapCities(city, cityMapping)
    return city
```

Once the city names were clean, I could run a query to observe the amount of entries for each city


```python
df1 = pd.read_sql_query('SELECT tags.value, COUNT(*) as count \
                        FROM (SELECT * FROM nodes_tags UNION ALL \
                              SELECT * FROM ways_tags) tags \
                        WHERE tags.key LIKE  "city" \
                        GROUP BY tags.value \
                        ORDER BY count DESC;', conn)
```

This output the following data for the first five entries:

|value|count|
|-----------|
|Denver|29522|
|Lafayette|3684|
|Boulder|3263|
|Aurora|1915|
|Broomfield|1005|
Overall there was a total of 80 unique cities. The bulk being in the largest city Denver, but also gathering data from quite a few neighboring towns in the area.

### Cleaning Phone Numbers

The phone numbers required some more extensive cleaning than the street and city names. I wanted all of the data to be in the same format to be easy to read. Looking through the data, some of the numbers were entered with two numbers but not as separate values. By using a short regular expression:


```python
findMultiples = re.compile(r'[\;]')
match = re.search(findMultiples, phone)
```

I was able to find any instances where two numbers were in the same entry and separate them. Then, I decided that any numbers were less than 10 characters were not valid and shouldn't be included with the data so the invalid numebers were removed. By using another regular expression: 


```python
correctFormat = re.compile(r'^(\+1) /(\d{3}/) \d{3}-\d{4}')
```

I could find any data that had the correct format that I wanted and return it. If the number was not in this format, I removed any characters except for the numbers. This made it easy to then put each number into the same format with the following code:


```python
def standardize(phone):
    # Remove everything but the number
    phone = [item.replace('(', '').replace(
        ')', '').replace(' ', '').replace('-', '').replace('.', '')
        for item in phone]
    phone = "".join(phone)
    # Put numbers to same format
    if phone.startswith('+01'):
        phone = '+1 ' + '(' + str(phone[3:6]) + ')' + ' ' \
                + str(phone[6:9]) + '-' + str(phone[9:])
    elif phone.startswith('+1'):
        phone = '+1 ' + '(' + str(phone[2:5]) + ')' + ' ' \
                + str(phone[5:8]) + '-' + str(phone[8:])
    elif phone.startswith('1'):
        phone = '+1 ' + '(' + str(phone[1:4]) + ')' + ' ' \
                + str(phone[4:7]) + '-' + str(phone[7:])
    else:
        phone = '+1 ' + '(' + str(phone[:3]) + ')' + ' ' \
                + str(phone[3:6]) + '-' + str(phone[6:])
    return phone
```

## Data Overview

### File Sizes

 - denver-boulder_colorado.osm : 956 MB

 - mapData.db : 79 MB

 - nodes_tags.csv : 14.6 MB

 - nodes.csv : 374.4 MB

 - ways_nodes.csv : 119.8 MB

 - ways_tags.csv : 60.8 MB

 - ways.csv : 30.2 MB


### Number of Unique Users



```python
df2 = pd.read_sql_query('SELECT COUNT(DISTINCT(x.uid)) \
                         FROM (SELECT uid FROM nodes UNION ALL \
                               SELECT uid FROM ways) x;', conn)
print df2
```

2294

### Number of Nodes


```python
df3 = pd.read_sql_query('SELECT COUNT(*) FROM nodes;', conn)
print df3
```

4360962

### Number of Ways


```python
df4 = pd.read_sql_query('SELECT COUNT(*) FROM ways;', conn)
print df4
```

487537

### Top 10 Contributors


```python
df5 = pd.read_sql_query('SELECT x.user, COUNT(*) as count \
                         FROM (SELECT user FROM nodes UNION ALL \
                               SELECT user FROM ways) x \
                         GROUP BY x.user \
                         ORDER BY count DESC \
                         LIMIT 10;', conn)
print df5
```

|user|count|
|---------------|
|chachafish|823201|
|Your Village Maps|734747|
|woodpeck_fixbot|340630|
|GPS_dr|313217|
|jjyach|309141|
|DavidJDBA|186559|
|Stevestr|170228|
|CornCO|156123|
|russdeffner|124346|
|Berjoh|84235|

### Number of users with less than 5 posts


```python
df6 = pd.read_sql_query('SELECT COUNT(*) \
                         FROM (SELECT x.user, COUNT(*) as num \
                         FROM (SELECT user FROM nodes UNION ALL \
                               SELECT user FROM ways) x \
                         GROUP BY x.user \
                         HAVING num < 10);', conn)
print df6
```

950

## Additional Statistics

With a total number of 4,848,499 entries, the top user chachafish contributed approximately 17%. It would appear that the spread of entry values is decently distributed. However with such a large amount of unique users, the bulk of entries are kept for the top 10 users. Overall the top 10 users account for near 67% of all entries. This means there are nearly 2,000 users contributing very little. Almost half of the users have less than 10 posts.

I play a good amount of video games in my spare time. Observing the top 10 contributors table, to me it looks very similar to a leaderboard in a video game. To give some incentive for more posts, all it could take is to display that graph publicly somewhere on the open street map site. Any form of competition typically drives people to be at the top, even if it is just showing who the top contributors currently are.

## Ideas for Improvement

Building on the idea of a video game type leaderboard from the top user query, it may be useful to combine the data cleaning code with an actual videogame. The first one that comes to mind would be the popular Pokemon Go app that so many people use. If the developers could add a sort of "Where did you find this Pokemon?" option when players catch certain Pokemon, that could be linked to the open map data. The players could get some points for incentive to find new areas. There are advantages and disadvanteges to this however:

|Advantages|Disadvantages|
|-----------|-------------|
|Create content very quickly|Possible wrong data|
|Data from a wide range of areas|May drain phones battery quicker|
|More users instead of bots|Areas without players would lack data|

### Top 10 Amenities


```python
df7 = pd.read_sql_query('SELECT tags.value, COUNT(*) as num \
                         FROM (SELECT * FROM nodes_tags UNION ALL \
                               SELECT * FROM ways_tags) tags  \
                         WHERE key = "amenity" \
                         GROUP BY tags.value \
                         ORDER BY num DESC \
                         LIMIT 10;', conn)
print df7
```

|value|num|
|--------|
|parking|16809|
|restaurant|2294|
|school|1507|
|fast_food|1075|
|bench|963|
|place_of_worship|959|
|bicycle_parking|919|
|fuel|752|
|shelter|622|
|bank|543|

### Top 5 Religions


```python
df8 = pd.read_sql_query('SELECT nodes_tags.value, COUNT(*) as num \
                         FROM nodes_tags \
                         WHERE nodes_tags.key="religion" \
                         GROUP BY nodes_tags.value \
                         ORDER BY num DESC \
                         LIMIT 5;', conn)

```

|value|num|
|----------|
|Christian|573|
|Jewish|9|
|Buddhist|4|
|Muslim|3|
|Multifaith|2|

### Streets to live on if you like Mexican food


```python
df8 = pd.read_sql_query('SELECT nodes_tags.value, COUNT(*) as num \
                         FROM nodes_tags \
                         JOIN (SELECT DISTINCT(id) FROM nodes_tags \
                               WHERE value = "mexican") x \
                         ON nodes_tags.id = x.id \
                         WHERE nodes_tags.key="street" \
                         GROUP BY nodes_tags.value \
                         ORDER BY num DESC \
                         LIMIT 5;', conn)
```

|value|num|
|-----------|
|East Colfax Avenue|14|
|      East Hampden Avenue    |5|
|       Colorado Boulevard    |4|
|  East Mississippi Avenue    |4|
|        South Parker Road    |4|

### Streets to live on if you like Beer


```python
df9 = pd.read_sql_query('SELECT nodes_tags.value, COUNT(*) as num \
                         FROM nodes_tags \
                         JOIN (SELECT DISTINCT(id) FROM nodes_tags \
                               WHERE value = "pub") x \
                         ON nodes_tags.id = x.id \
                         WHERE nodes_tags.key="street" \
                         GROUP BY nodes_tags.value \
                         ORDER BY num DESC \
                         LIMIT 5;', conn)
print df9
```

 |value|  num|
 |--------|
|      Market Street  |  8|
|       Blake Street   | 6|
|    Tennyson Street    |4|
|  East Iliff Avenue    |3|
|        15th Street    |2|

## Conclusion

After cleaning and reviewing the data, it is apparent that there is still a large bulk of data that could use formatting. Further cleaning of the phone numbers could include separating the extensions from the actual phone number. Various beginnings of street names were not taken into consideration for the scope of this project however that should be the next focus for future improvements. Most, if not all of the top users appear to be programmed bots. If whatever programming the bots use was merged with the code for this project, the data could be input correctly in the first place. However then comes the difficulty of all the bots having the same format. Overall, the data that was cleaned proved useful for the scope of this project. I beleive if I ever do decide to move to Denver, I will most likely be living on Market Street.

