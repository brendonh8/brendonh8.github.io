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