## Identiy Fraud From Enron Email Data

### Overview

In 2000, a company named Enron was one of the largest companies in the United States. Based in Houston, they became one of the worlds major energy and commodoties companies. Corporate fraud spread throughout the company and eventually led to their downfall, causing them to collapse into bankruptcy in 2002. In the resulting Federal investigation, a significant amount of typically confidential information entered into the public record, including tens of thousands of emails and detailed financial data for top executives.

>Summarize for us the goal of this project and how machine learning is useful in trying to accomplish it. As part of your answer, give some background on the dataset and how it can be used to answer the project question. Were there any outliers in the data when you got it, and how did you handle those?

The goal of this project is to use that financial and email data that was made available to determine if an employee has committed fraud. These people will be known as POI's(persons of interest). This means they are in one of the following categories: indicted, settled or plea deal with the government, or testified in exchange for prosecution immunity.

## Data Exploration

| Description | Result |
|----------- | ------|
|Number of People| 146|
|Number of People of Interest| 18 |
|Number of Features | 21 |

With a total of 146 people, each person in the dataset has 21 different features that might be used to determine whether a person is a POI or not. These features can be split into three categories; financial, email, and if they were actually a POI. These features are:
target_label = 'poi'

**Email Features**: `[to_messages, email_address, from_poi_to_this_person, from_messages, from_this_person_to_poi, shared_receipt_with_poi]`

**Financial Features**: `[salary, deferral_payments, total_payments, loan_advances, bonus, restricted_stock_deferred, deferred_income, total_stock_value, expenses, exercised_stock_options, other, long_term_incentive, restricted_stock, director_fees]`

**Target Label**: `[poi]`

### Missing Data






