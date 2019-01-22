# Predicting Customer Churn

The goal of this repository is to show how we can predict customer churn with Spark.
Although we are using Spark in local mode here, and technically the data could be 
analyzed on a single machine, we process the data and build the model with Spark in order
to create an extensible framework. The analysis conducted here could scale to much
bigger datasets, provided the code be deployed on a cluster capable of handling the 
computations necessary. 

Sparkify is an imaginary music app company, and we use a small subset of their log
data to try and predict churn. More context on this analysis and interpretation of
the results can be found in [this blog post on Medium](https://medium.com/@celestinhermez/predicting-customer-churn-with-spark-4d093907b2dc).

## File Structure

* **mini_sparkify_event_data.json**: this is the log data that we use in this example.
It contains information about 226 distinct users, with actions as detailed as giving
a thumbs up to a song or changing the settings of the account
* **Sparkify.ipynb**: this is the Spark notebook which contains all the code for this 
analysis. In order to run it, `pyspark`, `pandas`, `matplotlib`, `seaborn` and
`datetime` have to be installed
* **Sparkify.html**: an HTML version of the notebook
* **images**: the three png files are the graphs included in the Medium blog post

## Analysis

After loading and cleaning the data, we create features, both related to the nature of the 
account (paid vs. free, state, registration date) and to behaviors taken on platform
(thumbs up, creation of playlists, number of songs per session etc.). We engineer
these features with Spark and leverage the `Pipeline` class to efficiently process the data.
One processing step which is particularly important is to upsample the positive class
since the proportion of users who churned is small and this could bias the accuracy 
of our model. To do so, we sample with replacement from the population of users who
churned.

We then test out various classification models (`LogisticRegression`, `RandomForestClassifier`,
`GBTClassifier`) and compare their accuracy and F1 score on the test set. From there,
we choose to tune a logistic regression model further through a `CrossValidator` 
performing the GridSearch algorithm on 3 folds, with the accuracy as the optimization
metric. We choose to tune the logistic regression because of the interpretability of 
its coefficients. 

After hyperparameter tuning, we have a model which has 73% accuracy on the test set,
with a F1 score of 0.7. This means we are not disproportionately predicting one of the classes and
have a good balance between precision and recall. Interestingly, there is no improvement
after hyperparameter tuning through Grid Search, most likely due to the small size
of our train set (191 users). 
Examining feature importance, we conclude that both static features (such as the state
or the registration month) as well as on-platform behaviors (adding a friend) matter
when making a prediction. Getting this insight is crucial for Sparkify, because they can
target these users at risk with special offers (discounts, personalized messages) to try  
and mitigate the churn which is likely to happen. Internally, this could be automated
by running the model regularly (every day/week) and flagging users at risk.

## Possible Improvements

This analysis would gain from leveraging a larger dataset and being deployed on a cluster.
Grid search is a particularly computationally expensive operation, but with larger resources and more time
a more extensive search over a larger dataset and hyperparameter space
could be conducted to further tune the model and likely improve overall accuracy.

Moreover, this model should not be static but run somewhat regularly as user behaviors
and the consumer base evolve. It is important not to rely on an outdated model for such
an important aspect of the business.

Finally, some A/B tests could be set up to examine the insights of this model, 
in particular the resulting mitigating actions taken. One possibility would be 
to find users identified as potential churners, split them into
a control and treatment group, assign some "churn-mitigating" treatment and compare their 
churn rates through statistical hypothesis testing. This approach would be a rigorous
follow-up to this model. 