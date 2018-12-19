---
title: "Instacart Kaggle Competition"
date: 2018-10-31
tags: [Python, AWS, sklearn, plotly]
htmlwidgets: TRUE
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

<div>
    <a href="https://plot.ly/~brendonh8/9/?share_key=qliPLVkzwR1mnLPhbnXj9h" target="_blank" title="Plot 9" style="display: block; text-align: center;"><img src="https://plot.ly/~brendonh8/9.png?share_key=qliPLVkzwR1mnLPhbnXj9h" alt="Plot 9" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="brendonh8:9" sharekey-plotly="qliPLVkzwR1mnLPhbnXj9h" src="https://plot.ly/embed.js" async></script>
</div>

Looking further into the orders throughout the day makes it clear that Saturday and Sunday have the largest order numbers. There are also small spikes in the morning and evening times.

<div>
    <a href="https://plot.ly/~brendonh8/7/?share_key=vQl8lWC3iRPF1DqJECXJhq" target="_blank" title="Plot 7" style="display: block; text-align: center;"><img src="https://plot.ly/~brendonh8/7.png?share_key=vQl8lWC3iRPF1DqJECXJhq" alt="Plot 7" style="max-width: 100%;width: 600px;"  width="600" onerror="this.onerror=null;this.src='https://plot.ly/404.png';" /></a>
    <script data-plotly="brendonh8:7" sharekey-plotly="vQl8lWC3iRPF1DqJECXJhq" src="https://plot.ly/embed.js" async></script>
</div>

The order trends that customers have seem to be on a week to week basis. There are some clear spikes in orders around every 7 days since their last order. The spike at 30 is most likely due to customers that did not order again. 

Most of the products that customers order tend to be produce. Apparently Instacart customers love Bananas. 

![image-center](/assets/images/instacartproject/top_products.png)

Keep in mind, I used a fraction of the full dataset to perform EDA, the actual dataset was too large to output these plots quickly. The general trend of the data is the same.

## <a name="heading-3"></a>Feature Engineering

Initially I kept using the smaller dataset to for features, however when I actually tested the features and started modeling I needed to switch to AWS. My 8 GB Macbook started to have some trouble with the full 3 million orders.

I kept track of my feature progress by creating a baseline and continuously adding and testing features to see what would help the model. The first thing to do was actually create the target. I needed to predict whether or not a product will be ordered in the next cart, so I used the most recent orders to create a new column for each product of whether or not it is in the most recent cart. This 0 or 1 value is what I will try to predict. This also means that the feature table needs to be made up of user-product combinations, not order numbers. I did this with a short script below.

```python
def get_latest_cart(x):
    cart = {'latest_cart': set(product for product in x['product_id'])}
    return pd.Series(cart)

train_carts = (df_order_products_train.groupby('user_id').apply(get_latest_cart)
                                                        .reset_index())
df_X = df_X.merge(train_carts, on='user_id')
df_X['in_cart'] = (df_X.apply(lambda row: row['product_id'] in row['latest_cart'], axis=1).astype(int))
```

Pandas makes it easy to apply functions to dataframes this way. This is how I created many of my features as well. My first feature was the total times a user has ordered a product.