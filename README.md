# Predicting Customer Churn

The goal of this repository is to show an example of predicting customer churn with Spark.
Although we are using Spark in local mode here, and technically the data could be 
analyzed on a single machine, we process the data and build the model with Spark in order
to create an extensible framework. The analysis conducted here could scale to much
bigger datasets, provided the code be deployed on a cluster capable of handle the 
computations necessary. 

Sparkify is an imaginary music app company, and we use a small subset of their log
data to try and predict churn. More context on this analysis and interpretation of
the results can be found in this blog post on Medium (link).

## File Structure

* **mini_sparkify_event_data.json**: this is the log data that we use in this example.
It contains information about 226 distinct users, with actions as detailed as giving
a thumbs up to a song or changing the settings of the account
* **Sparkify.ipynb**: this is the Spark notebook which contains all the code for this 
analysis

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
its coefficients. In addition, our optimization metric is the accuracy rather than the
F1 score because we have already adjusted for the class imbalance, and as such there
is less of a disconnect between accuracy and F1 score.

After hyperparameter tuning, we have a model which has ?? accuracy on the test set,
with a F1 score of ??. This means we are not overly predicting one of the classes and
have a good balance between precision and recall. Interestingly, looking at the coefficients
we conclude that ???. Getting this insight is crucial for Sparkify, because they can
target these users at risk with special offers (discounts, personalized messages) to try  
and mitigate the churn which is likely to happen. Internally, this could be automated
by setting up automated alerts once users hit certain thresholds on ?? variable for instance.

## Possible Improvements

This analysis would gain from leveraging a larger dataset and being deployed on a cluster.
Grid search is a particularly expensive operation, but with larger resources and more time
a more extensive search could be conducted to further tune the model and likely improve
overall accuracy.

Moreover, this model should not be static but run somewhat regularly as user behaviors
and the consumer base evolve. It is important not to rely on an outdated model for such
an important aspect of the business.

Finally, some A/B tests could be set up to examine the impact of this model. One
possibility would be to find users identified as potential churners, split them into
a control and treatment group, assign some "churn-mitigating" action and compare their 
churn rates. 