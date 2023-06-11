<style>
</style>

# Power Outage Cause Classifier
*Authors: Garvey Li, Penny King*

## Introduction

Our project focuses on predicting the cause of a power outage. Sometimes power outages may occur due to unexpected reasons with the cause for it being unclear. Our project aims to build a classiifer using readily avaialble information you can get shortly after a power outage. This classification model may be useful to possibly figure out why a specific area is getting power outages when they are unaware of why they are happening and to mitigate that issue and prevent future outages from happening in the future.

We decided to tackle this with a multiclass classification model, specifically a Decision Tree Classifier. We will be predicting `CAUSE.CATEGORY`, which corresponds to the reason why the power outage occured. We chose a Decision Tree Classifier specifically because it performs well with multiclass classification problems as well as high dimension data. Since this dataset is very robust with many features, we decidied to go with this model as our focus.

For our evaluation metric we used F1-score because our respones variable (`CAUSE.CATEGORY`) is very imbalanced. Due to this imbalance in our data we decided that F1-score was a better choice over other metric such as accuracy because of how it balances precision and recall.

We trained out model only using these following columns:
* `U.S._STATE`
* `CLIMATE.CATEGORY`
* `MONTH`
* `SEASON`
* `OUTAGE.DURATION`
* `POPDEN_URBAN`
* `POPPCT_URBAN`
* `POPPCT_UC`
* `AREAPCT_URBAN`
* `AREAPCT_UC`
* `PCT_LAND`
* `PCT_WATER_TOT`
* `PCT_WATER_INLAND`
* `NERC.REGION`
* `CLIMATE.REGION`

The reason why we chose to predict using these features in particular were because these would likely be the known information we have at the "time of prediction" as these features are data that you can easily access post-power outage if you did not know the cause. For example, location specific data such as the state, climate category, the population metrics (urban and rural population, etc) are readily available even before the power outage arises. And once the power outage ends, the duration of the outage is readily accessible information.

## Baseline Model

Our initial baseline model we created was a Decision Tree Classifier with the qualitiatve variables of:
* `U.S._STATE` (nominal)
* `CLIMATE.CATEGORY` (nominal)
* `MONTH` (ordinal)
* `NERC.REGION` (nominal)
* `CLIMATE.REGION` (nominal)
All qualtitative variables we One Hot Encoded before putting into our model.

*(!) check OneHotEncoding after TA responds!*

and the quantitative data we used was:
* `OUTAGE.DURATION`
* `POPDEN_URBAN`
* `POPPCT_URBAN`
* `POPPCT_UC`
* `AREAPCT_URBAN`
* `AREAPCT_UC`
* `PCT_LAND`
* `PCT_WATER_TOT`
* `PCT_WATER_INLAND`


The hyperparameters we got from GridSearchCV (grid search cross validation with 5 folds) for our baseline Decision Tree Classifier was:
* `criterion`: gini
* `max_depth`: 3
* `min_samples_split`: 200


*(!) MAY NEED TO ADD MORE! project says many ppl lose points here*
The performance of our model on our test data was an F1-score of 0.5875980924470244. This is not very high, as values closer to 1.0 mean a better F1-score. Having an F1-score over 0.5 and closer to 0.7 would be a better F1-score metric. This lower F1-score means that our model does not have a high precision or recall rate. This means that our model is not accurate in predicting true positive predictions and is not generally specific in its predictions. 


## Final Model

*State the features you added and why they are good for the data and prediction task. Note that you can’t simply state “these features improved my accuracy”, since you’d need to choose these features and fit a model before noticing that – instead, talk about why you believe these features improved your model’s performance from the perspective of the data generating process.*

In our final model the features we included were:

Qualitiatve variables:
* `U.S._STATE` (nominal)
* `CLIMATE.CATEGORY` (nominal)
* `MONTH` (ordinal)
* `NERC.REGION` (nominal)
* `CLIMATE.REGION` (nominal)
* `SEASON` (nominal)
All qualtitative variables we One Hot Encoded before putting into our model.

*(!) check OneHotEncoding after TA responds!*

Quantitative data we used was:
* `OUTAGE.DURATION`
* `POPDEN_URBAN`
* `POPPCT_URBAN`
* `POPPCT_UC`
* `AREAPCT_URBAN`
* `AREAPCT_UC`
* `PCT_LAND`
* `PCT_WATER_TOT`
* `PCT_WATER_INLAND`


We feature engineered a new column `SEASON` which we got from the `MONTH` column in our DataFrame.
We also created quantiles for all our quantitaive data. The reason why we decided to do these instances of feature engineering was to group our data. We noticed that our data in many of these columns were very spread out and unique, which may be causing issues for our decision tree model when it comes to classifying specific classes.

Creating quantiles helps with giving data class at the extremes and the middle the same number ovalues. this qual representation may be beneficial for the model to learn the patterns associated with the class and can help give better ways for the decision tree to split.

Grouping by seasons also helps with splitting within the decision tree classifier.

We decided to find the optimal hyperparameters for our model using 5-fold grid search cross validation from `sklearn` package within our modeling Pipeline.
The hyperparameters we got from GridSearchCV (grid search cross validation with 5 folds) for our final Decision Tree Classifier was:
* `criterion`: gini
* `max_depth`: 7
* `min_samples_split`: 50

This new model with new hyperparameters and features resuled in a F1-score performance of 0.6372507104664724. This is a significant improvement from our baseline model, which was 0.5875980924470244. This means that our model improved in its precision and recall (specificity and sensetivity), but not but too much of significant amount to be truly reliable just yet. 

*Optional: Include a visualization that describes your model’s performance, e.g. a confusion matrix, if applicable.*

| Actual                        |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |\n|:------------------------------|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|\n| equipment failure             |                       1 |                   10 |           1 |               6 |               16 |                               8 |\n| fuel supply emergency         |                       9 |                    4 |           0 |               0 |               23 |                               1 |\n| intentional attack            |                       1 |                  270 |           2 |               4 |               42 |                               3 |\n| islanding                     |                       0 |                   12 |           6 |               0 |                5 |                              12 |\n| public appeal                 |                       0 |                    6 |           1 |              21 |               21 |                               2 |\n| severe weather                |                       5 |                   32 |           2 |              13 |              485 |                              16 |\n| system operability disruption |                       1 |                   21 |           0 |               6 |               45 |                              30 |

## Fairness Analysis

*Clearly state your choice of Group X and Group Y, your evaluation metric, your null and alternative hypotheses, your choice of test statistic and significance level, the resulting p-value, and your conclusion.*

*Optional: Embed a visualization related to your permutation test in your website.*

*Tip: When making writing your conclusions to the statistical tests in this project, never use language that implies an absolute conclusion; since we are performing statistical tests and not randomized controlled trials, we cannot prove that either hypothesis is 100% true or false.*




