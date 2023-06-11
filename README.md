# Modeling based on Data of Power Outages in the United States from 2000 - 2016

## Framing the Problem

My exploratory data analysis on this dataset can be found [here](https://dante-t0.github.io/power-outages-analysis/). 
This includes a more in depth description of the dataset, and the correlations that are contained within.

Here, I created two models based on this dataset with the goal to predict the *duration* of a given power outage. These two models differ in the amount of information that is assumed the user has. The first basic model is based on basic location metrics that one would have immediately following an outage. The second model uses more complex features such as cause and breadth of outage, which would take some more time to get. All of these features are rooted on information that one would have prior to the outage ending, and therefore before the duration of said outage would have been finalized. 

I chose this response variable (Outage Duration) because it is the value that appears last, and an important aspect of a power outage. Additionally, the exploratory data analysis appeared to show correlation between duration and various geographical features. 

To measure the success of the models, I will be comparing R^2 score and Root Mean Squared Erorr (RMSE).

##  Baseline Model

This first model is based on the following features, undergoing the following transformations:
- 'U.S._STATE' : The U.S. State where the outage occured, One Hot Encoded (*Nominal*)
- 'NERC.REGION' : The geographical region designated by the NERC (North American Electric Reliability Corporation), One Hot Encoded (*Nominal*)
- 'CLIMATE.REGION' : The climate region the outage occured (e.g West, Northeast, Central, etc.), imputed with the most frequent region, One Hot Encoded (*Nominal*)
- 'CLIMATE.CATEGORY' : The climate category the outage occured (e.g. normal, cold, warm), imputed with the most frequent region, One Hot Encoded (*Ordinal*)

These features are all factored into a model that uses a distance weighted 'K Neighbors Regressor' which takes uses the nearest 10 neighbors, using the inverse of their distances, of any given point to predict the Outage Duration. The inverse distances are used to minimize the effect of outliers.

The model was trained on a 'train' subsection of the 2000 - 2016 dataset, imputing the outage duration with the median of the total dataset.

Testing the model on a 'test' subseciton of the dataset results in a R^2 score of -0.867083843620877 and a RMSE of 6781.52647286401

The R^2 score gives us a general sense of how 'good' this model is, which, is not very good. The fact that the score is negative shows that this model is worse than merely predicting the duration of every outage with the mean of the train outages. This indicates that this model needs more work. The RMSE score means nothing currently, as instead it acts as a comparison for future, more complex models. 

## Final Model

The final model is based on all the features from the baseline model, and the following *new* features, undergoing the following transformations:
- 'FROM_HURRICANE' : Whether or not the outage originated from a Hurricane, created from a column called 'HURRICANE.NAMES', representing binarily if it had a name in this column (1 - originated from a hurricane; 0 - did not originate from a hurricane) (*Quantitative (Binary) originated from a Nominal Column*)
- 'CAUSE.CATEGORY' : The general cause of the outage (e.g. severe weather, intentional attack, etc.), One Hot Encoded (*Nominal*)
- 'CUSTOMERS.AFFECTED' : The breadth of the outage measured in number of customers affected, imputed with the median number of customers affected, Transformed using quantile information with 750 quantiles. (*Quantitative*)

These features are chosen because they give a lot more information than merely the geographic and climate information in the baseline model. These are also features that, theoretically, should directly affect how long the outage continues for.

All of these features were factored into a model that uses a Random Forest Regressor, which uses a variety of decision trees with differing depths to predict the Outage Duration. This specific Regressor has 2 hyperparamters, which were tested using GridSearchCV, which tests a variety of values for the hyperparameters to find the best one. The following is the results of that search:
- 'n_estimators' : the number of trees in the forest; Looking at hyperparameter values from 2 to 482, in steps of 20, the 'ideal' number of trees was found to be 262.
- 'max_depth' : the maximum depth of each tree; Looking at hyperparameter values from 2 to 20, the 'ideal' max depth was found to be 3. 

The model was trained on the same 'train' subsection of the 2000 - 2016 dataset, imputing the outage duration with the median of the total dataset.

The results, compared to the baseline, are the following:
| model    |   R-Squared |    RMSE |
|:---------|------------:|--------:|
| baseline |   -0.867084 | 6781.53 |
| final    |    0.231066 | 4352.01 |

Both metrics showed great improvement from the baseline, with the RMSE showing a **35.8%** decrease. The R^2 shows that the model is a good amount better than merely using the mean, but it is not good enough to reliably predict duration. 