# Recipes-Rating-Analysis
a project for DSC 80 at UCSD Analyzing a database of recipes and their evaluations

by: Jiawei Lyu

## Introduction

As the number of recipes grows, so does the difficulty of choosing the right one. Especially nowadays, when people are busy and have high obesity rates, they need recipes that are quick and not too high in calories. In this study, I will focus on recipes that can be made in less than two hours with not too high calories and predict if the recipe will take less than one hour based on the information in the recipe.

The dataset is from food.com which contains recipes and reviews. The recipe dataset has important inforamtion realated to our question like id (Recipe ID), n_steps(number of steps), minutes(Minutes to prepare recipe), tags(Food.com tags for recipe), and nutrition(Nutrition information). It has 83782 rows. The ratings dataset has inforamtion realated to our question like recipe ID, and rating and it has 731927 rows.


## Data Cleaning and EDA

### Data cleaning

First, since this study will only focus on recipes that are less than two hours old, I'll be processing the data, removing unneeded columns and adding some columns that make sense.

For the interacts database, I only kept the recipe id and rating because it will eventually be used to calculate the average rating of the recipes.


For the recipes database, first, I converted all the values in the dataset that should be lists, but are actually strings, into lists. then, I added two new columns : the number of tags and the calorie value. Finally, I removed the unneeded columns : "contributor_id", "tags", "nutrition", "steps", "description", "ingredients" (They were removed because they were partly difficult to use for data analysis and partly because I had already extracted useful information).Then I calculated the average score for each recipe and added it to the recipes dataset. Finally, I filtered out recipes that took more than two hours to make, as well as recipes with more than 1,500 calories. Finally, I split the minutes into categorical features, namely "0-59" and "60-120".


Finally, the cleaned dataset looks like:

| name                                 |     id | submitted   |   n_steps |   n_ingredients |   n_tags |   calories |   avg_rating | minutes_cat   |
|:-------------------------------------|-------:|:------------|----------:|----------------:|---------:|-----------:|-------------:|:--------------|
| 1 brownies in the world    best ever | 333281 | 2008-10-27  |        10 |               9 |       14 |      138.4 |            4 | 0-59          |
| 1 in canada chocolate chip cookies   | 453467 | 2011-04-11  |        12 |              11 |        9 |      595.1 |            5 | 0-59          |
| 412 broccoli casserole               | 306168 | 2008-05-30  |         6 |               9 |       10 |      194.8 |            5 | 0-59          |
| millionaire pound cake               | 286009 | 2008-02-12  |         7 |               7 |       20 |      878.3 |            5 | 60-120        |
| 2000 meatloaf                        | 475785 | 2012-03-06  |        17 |              13 |       10 |      267   |            5 | 60-120        |

### EDA

For EDA, first, I checked the distribution of all the features, and one interesting finding was that the avg_rating feature is heavily skewed (see figure below), with an extremely large number of recipes rated 5.0, so skewed that I don't think it provides any useful information for prediction purposes anymore. The skew is so bad that I don't think it provides any useful information when doing prediction. Therefore it will not be used in the prediction. And for the other features, although they are also skewed, we can make their distribution normal by taking log.

<iframe src="assets/uni_avg_rating.html" width=800 height=600 frameBorder=0></iframe>

Also important is the distribution of minutes_cat (minute catgories), which we can see is unbalanced. (see figure below) This will affect the choice of metric when we do the prediction later.

<iframe src="assets/uni_min_cat.html" width=800 height=600 frameBorder=0></iframe>

Then, for the bivariate analysis, I analyzed the relationship between "n_steps", "n_ingredients", and "calories". None of the three figures (see below for an example of the relationship between steps and ingredients) show a strong correlation. This is what we would expect, as we do not want to be influenced by multicollinearity when we make our predictions.

<iframe src="assets/bi_step_ing.html" width=800 height=600 frameBorder=0></iframe>

Finally I did aggregation on feature "tags" and calculated its max, mean, and median for different time classifications, and found it to be almost identical respectively. (see figure below) This likely indicates that it provides no meaningful information for prediction.

<iframe src="assets/agg_tag_min.html" width=800 height=600 frameBorder=0></iframe>

## Assessment of Missingness

### NMAR Analysis

(For this part only, I will use the unfiltered dataset, and I didn't drop any columns)

. We can see that this dataset has 3 features containing NA values. Name, Description, and Avg_rating. I think that for Name, the NA value generated is most likely **NMAR**, probably because: some of the text is from a foreign language and can't be decoded with the current decoder. To prevent the program from crashing, the names are changed to NaN. I might be able to explain missingness a little better if I could know that all these recipes were uploaded by people in those countries.

| index          |           0 |
|:---------------|------------:|
| name           | 1.19357e-05 |
| id             | 0           |
| minutes        | 0           |
| contributor_id | 0           |
| submitted      | 0           |
| tags           | 0           |
| nutrition      | 0           |
| n_steps        | 0           |
| steps          | 0           |
| description    | 0.000835502 |
| ingredients    | 0           |
| n_ingredients  | 0           |
| avg_rating     | 0.0311403   |

### Missingness Dependency

And for avg_rating, I think it's likely to be MAR because a late submitted recipes will get fewer reviews and ratings compared to early submitted recipes, and therefore more likely to get NA value. We split the database into two parts, one where "avg_rating" is missing and the other where it is not, and we can see that the distribution of ids is not similar. We can see that for recipes where avg_rating is not NA, they have more ids that are small (i.e. submitted early).

<iframe src="assets/mis_id_rate.html" width=800 height=600 frameBorder=0></iframe>

I then performed a permutation test to randomly permute the ids of the recipes and obtained the distribution of the K-S Statistic. as can be seen, the p_value of the values that we observe for the K-S Statistic is essentially 0. Therfore, the conclusion is I reject the null hypothesis, and it's likely that the empty value in avg_rating is **MAR**

<iframe src="assets/fig_ks_id.html" width=800 height=600 frameBorder=0></iframe>

## Hypothesis Testing

Now, I'm going to perform a Hypothesis test. 

The Null hypothesis is: Recipes that take less time (<1 h) and those that take more time (>= 1 h) have the same steps. 
Alternative hypothesis is: Recipes that take less time (<1 h) have higher steps than those that take longer (>= 1 h). 

I will perform permutation test. test statistic is difference in mean of n_steps. significance level is 0.05.

<iframe src="assets/fig_step4.html" width=800 height=600 frameBorder=0></iframe>

As can be seen, it is almost impossible to see this distribution simply because of randomness. p-value is almost 0. The conclusion is I reject the null hypothesis, and it's likely that recipes that take less time (<1 h) have higher steps than those that take longer (>= 1 h). The significance of understanding this is that we can assume that the feature n_steps can be of great help when doing prediction. This is because it has a strong correlation with the time spent on the recipe.

## Framing a Prediction Problem

Now, we can get back to our most important question: does it take less than an hour to predict a recipe. This is a binary classification question(less than 1 hour or not), and the response variable I'm using is "minutes_cat" (minutes categories). This is the time category we want to predict. The metric I am using is F-1 score, since the categories are unbalanced, and we don't need to focus specifically on either side of recall or precision (making mistakes doesn't have serious consequences). What we know so far is that: "n_ingredients", "calories" and "n_steps" are independent of each other (as we mentioned in the bivariate analysis). So I would consider using them." The "n_tags" is a feature that I would not use because it has almost the same distribution across time categories (see aggregation in the EDA section).

## Baseline Model

The classifier I am using is a decision tree classifier. The features used so far are "n_ingredients" and "calories". They are quantitative features and since their distributions are skewed (see EDA part), I have performed a log transformation on them and I have not adjusted any hyperparameters.

The result is that the F-1score for the training set reached 0.97, but the F-1 score for the test set was only 0.86.Obviously, this mod has been overfitting. Therefore, I think this mod is not good.

## Final Model

The features I added are "n_steps" and "id". The reasons are: 1. "n_steps" is distributed independently of "n_ingredients" and "calories", so I would expect it to provide more information. 2. id provides the time difference between recipes, an information that the other three features don't provide. I also apply the log transformation on them.

The model I'm using is still Decision Tree Classifier. the reason is that even though it was overfitting before, it still has a good test F-1 score. Therefore, I think this model can reduce the effect of overfitting by adjusting the hyperparameters. By using GridSearchCV and providing different "min_samples_split" (5~35), "max_depth" (None~18), and different criterion (gini/entropy), I got the best hyperparameters: {'criterion': 'gini', 'max_depth': 4, 'min_samples_split': 5}.

Compared to before, the F-1 score for this mod in the training set is around 0.913, while the F-1 score for the test set is around 0.911. It is better than the previous mod in that it not only has a higher test F-1 score, but also has no overfitting.

## Step 8: Fairness Analysis

Finally, I will perform a fairness analysis on the produced model. The groupings I chose were: recipes with smaller id and recipes with larger id (split into two groups based on median value). evaluation metric is still F-1 score.

The Null hypothesis is: My model is fair. Its F-1 score for recipes with smaller id and recipes with larger id are roughly the same, and any differences are due to random chance. 
Alternative hypothesis is: My model is unfair. Its F-1 score for recipes with smaller id is lower than its precision for recipes with larger id. 

The test statistic is the difference in F-1 score between recipes with smaller id and recipes with larger id

<iframe src="assets/fig_fair.html" width=800 height=600 frameBorder=0></iframe>

As you can see from the images, it is almost impossible for us to see this difference due to randomness. The p_value is 0.006 So my conclusion is that I rejected the null hypothesis and it is possible that my mod is unfair.