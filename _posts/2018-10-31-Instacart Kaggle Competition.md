---
title: "Instacart Kaggle Competition"
date: 2018-10-31
tags: [Python, AWS, sklearn, plotly]
header:
    overlay_image: "/assets/images/instacartproject/burger.jpg"
excerpt: "Predicting the next item ordered using Instacart order data"
---
## Table of Contents

- [Overview](#heading-1)

- [Exploratory Data Analysis](#heading-2)

- [Feature Engineering](#heading-3)

- [Modeling](#heading-4)

- [Results/Conclusions](#heading-5)

- [Limitations/Future Work](#heading-6)

## <a name="heading-1"></a>Overview

Every week I bulk cook all of my meals so I tend to order the same things... a lot. If the grocery store wasn't so close to me, I would definately be an avid user of Instacart. Once I found their dataset for their Kaggle competition I figured it would be a perfect challenge for me. In case you have not heard of it, Instacart is a grocery delivery app. When you order from the app, someone will pick up your food from the grocery store and deliver it to you. The delivery persons credit card will be filled with just enough money to cover your groceries.

With all of these orders comes a lot of transactional data. In the Kaggle competition, Instacart wants to use this data to predict which previously ordered product will be in a user's next order. The competition link can be found [here](https://www.kaggle.com/c/instacart-market-basket-analysis)

## <a name="heading-2"></a>Exploratory Data Analysis

This is one of the largest data sets that I have been able to use so far. It contained transactional information for over 3 million orders made up of 5 thousand products. Eventually I had to use an AWS to engineer features and model. This dataset is a relational database made up of six tables. 

- **Orders**: The orders table has one order per row. The orders are split into past orders labeled as "prior" and the most recent orderes labeled as "train". Each order has info for the day of week ordered, time of day, and days since the previous order. 
- **Order Products Prior**: Gives information about which products were ordered for a certain order. It also  shows the add to cart order, and if the product is a reorder. These orders will be used to create features.
- **Order Products Train**: Similar to prior orders table except to be used for training.
- **Products**: Names of products and corresponding product id, aisle id, and department id.
- **Aisles**: Aisles and aisle id.
- **Departments**: Departments and department id.

In order to create a model that predicts if a product will be ordered next or not, features need to be created. Intuitively, the best features would be ones that explain human behavior. The best way to do that is first get an idea of the trends of orders.

![image-center](/assets/images/instacartproject/spread_over_week.png)

The daily trends show Saturday (0) and Sunday (1) being the days with the most orders. A majority of the orders are made between the morning and afternoon. 
