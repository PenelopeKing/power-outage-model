<style>

.table-wrapper {
    overflow-x: scroll;
}

</style>

# Power Outage Cause Classifier
*Authors: Garvey Li, Penny King*


Our exploratory data analysis project can be found [here](https://garveyjli.github.io/power_outage_data_exploration/).


## Introduction

Our project focuses on predicting the cause of a power outage. Sometimes power outages may occur due to unexpected reasons and the cause for it would be unclear. Our project aims to build a classiifer using information that is available shortly after a power outage. This classification model may be useful to possibly figure out why a specific area is getting power outages when they are unaware of why they are happening and to mitigate that issue and prevent future outages from happening in the future.

We decided to tackle this with a multiclass classification model, specifically a Decision Tree Classifier. We will be predicting `CAUSE.CATEGORY`, which corresponds to the reason why the power outage occured. We chose a Decision Tree Classifier specifically because it performs well with multiclass classification problems as well as high dimension data. Since this dataset is very robust with many features, we decidied to go with this model as our focus.

For our evaluation metric we used F1-score because our response variable,  `CAUSE.CATEGORY`, which contains 7 classes, has a very high class imbalance, as shown below. 

<iframe src="html_files/y_bar.html" width=800 height=600 frameBorder=0></iframe>

Due to this imbalance in our data we decided that F1-score was a better choice over other metric such as accuracy because of how it balances precision and recall.

We trained our model using the following features:
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


All qualitative variables were One Hot Encoded before putting into our model.


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


Below is the confusion matrix comparing the predicted class from our model and the actual class:


<div class="table-wrapper" markdown="block">


| Predicted                     |   fuel supply emergency |   intentional attack |   severe weather |
|:------------------------------|------------------------:|---------------------:|-----------------:|
 **Actual**                                                                                        |
| fuel supply emergency         |                       2 |                    1 |               10 |
| intentional attack            |                       0 |                   59 |               37 |
| islanding                     |                       0 |                    3 |                8 |
| public appeal                 |                       0 |                    4 |               14 |
| severe weather                |                       4 |                   10 |              192 |
| system operability disruption |                       0 |                    6 |               17 |
| equipment failure             |                       0 |                    3 |               12 |


</div>




We split our data into training and test set data to avoid overfit and also using a model’s performance on a test set is a good estimate of its ability to generalize to unseen data. When splitting we shuffled the data and used 25% of our data for testing and the rest of the 75% for training. This training and testing data will be used to fit and test both our baseline model and our finalized model.

The performance of our model on our test data was an F1-score of 0.5875980924470244. This is not very high, as values closer to 1.0 mean a better F1-score. Having an F1-score over 0.5 and closer to 0.7 would be a better F1-score metric. This lower F1-score means that our model does not have a high precision or recall rate. This means that our model is not accurate in predicting true positive predictions and is not generally specific in its predictions. In addition to this, the baseline model is not making any predictions of any other categories besides `fuel supply emergency`, `intentional attack`, and `severe weather`, meaning that only 3/7 classes are being predicted. This shows that the effects of the class imbalance present in the true data is significant here.


Bias and variance is a good weay to see how our model generalizes to unseen data:


The low F1-score indicates high deviation between predicted value and the actual value of the data. This may be a sign of underfit as the model is too basic. Looking at the hyperparameters, this may be due to `max_depth` = 3.


We performed 5-Fold train-test splitting and 5 times we trained our data and then tested it to find its f1_score on unique splits of data. This was how well the model perfromed after being fitted to testing data using different training data: [0.660127868111488, 0.5358888521716642, 0.5146355732018947, 0.6378973814389326, 0.537039896225877]. There seems to be a bit of deviation between predictions, which may be a sign of high variance. High variance can be due to the model overfitting to the training data. 




## Final Model

In our final model the features we included were:

Qualitative variables:
* `U.S._STATE` (nominal)
* `CLIMATE.CATEGORY` (nominal)
* `MONTH` (ordinal)
* `NERC.REGION` (nominal)
* `CLIMATE.REGION` (nominal)
* `SEASON` (nominal)
  

All qualitative variables we One Hot Encoded before putting into our model, as before in the baseline model.

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


We feature engineered a new column `SEASON` which we got from the `MONTH` column in our DataFrame. We believed that there may have been some correlations between the seasons and the distribution of outage causes for each season. For example, in the winter, there might be more strong storms and winds that can cause power outages as opposed to fall and spring, when the weather is more mild in comparison. Thus, there might be a higher probability of a power outage being caused by `severe weather` during winter as opposed to the fall or spring. So, grouping by seasons would help with splitting within the decision tree classifier.

We also created quantiles for all our quantitative data. The reason why we decided to do these instances of feature engineering was to group our data. We noticed that our data in many of these columns were very spread out and unique(high variance), which may be causing issues for our decision tree model when it comes to classifying specific classes. This representation may be beneficial for the model to learn the patterns associated with the class and can help give better ways for the decision tree to split.

We decided to find the optimal hyperparameters for our model using 5-fold grid search cross validation from `sklearn` package within our modeling Pipeline.
The hyperparameters we got from GridSearchCV (grid search cross validation with 5 folds) for our final Decision Tree Classifier was:
* `criterion`: gini
* `max_depth`: 7
* `min_samples_split`: 50

This new model with new hyperparameters and features resuled in a F1-score performance of 0.6372507104664724. This is a significant improvement from our baseline model, which was 0.5875980924470244. This means that our model improved in its precision and recall (specificity and sensitivity), but not too much of significant amount to be truly reliable just yet. 


Once again we look at the model's bias and variance to see how well it generalizes to unseen data:


The model has a higher F1-score now, meaning that the deviation between predicted and actual response variable is lower, meaning that the bias of the model decreased. This means the model is not underfitting as much. This makes sense given that after GridSearchCV our `max_depth` hyperparameter was 7. 

We performed 5-Fold train-test splitting and fit the model 5 times and tested it 5 times to see how the model varied in its F1-scores. This was how well the model perfromed after being fitted to testing data using different training data: [0.660127868111488, 0.5358888521716642, 0.5146355732018947, 0.6471442454979972, 0.537039896225877]. There seems to be a bit of deviation between predictions, which may be a sign of high variance. High variance can be due to the model overfitting to the training data. 


Below is the confusion matrix comparing the predicted class from our model and the actual class:


<div class="table-wrapper" markdown="block">


| Predicted                        |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|:------------------------------|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
|**Actual**                                                                                                      |
| fuel supply emergency         |                       9 |                    4 |           0 |               0 |               23 |                               1 |
| intentional attack            |                       1 |                  270 |           2 |               4 |               42 |                               3 |
| islanding                     |                       0 |                   12 |           6 |               0 |                5 |                              12 |
| public appeal                 |                       0 |                    6 |           1 |              21 |               21 |                               2 |
| severe weather                |                       5 |                   32 |           2 |              13 |              485 |                              16 |
| system operability disruption |                       1 |                   21 |           0 |               6 |               45 |                              30 |
| equipment failure             |                       1 |                   10 |           1 |               6 |               16 |                               8 |


</div>



In addition to having a higher F1 score, the effects of class imbalance are not as significant here compared to the baseline model, but still present. Only one category of cause was not predicted for this particular test sample(`equipment failure`)

## Fairness Analysis

We decided to analyze the fairness of our model in regards to the columns `POPPCT_URBAN`, which is the percentage of the population that is urban for the U.S. State in which the outage occurred. Looking at the distribution of percentages, there was a clear divide between a urban population percentage of ~70%, so we binarized it accordingly.

As a measure of fairness, we decided to use the absolute difference in F1 scores, since our data contained a lot of class imbalance, as mentioned before. The observed test statistic is the actual absolute difference between F1 scores of >=70% and <70% urban population, and the observed value was 0.04259. We conducted a permutation test using absolute difference in F1 scores as our test statistic.

With this information, our hypotheses are as follows:

**Null Hypothesis:** Our model is fair and its F1 score for outages that occured in populations that are less than 70% urban and populations that are
greater than or equal to 70% are roughly the same, and any differences are 
due to random chance with a significance level of 0.05.

**Alternative Hypothesis:** Our model is unfair and the F1 scores are different between populations that are less than 70% urban and populations that are greater than or equal to 70% are roughly the same, and any differences are NOT due to random chance with a significance level of 0.05

After running the permutation test with an observed statistic of 0.04259, we get a p-value of 0.58401, so we fail to reject the null hypothesis that our model is fair between populations that are less than 70% urban and populations that are greater than or equal to 70% urban.

<iframe src="html_files/fair_perm_test_3.html" width=800 height=600 frameBorder=0></iframe>




