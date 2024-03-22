# Recipes-Rating-Analysis
a project for DSC 80 at UCSD Analyzing a database of recipes and their evaluations
by: Jiawe Lyu

## Introduction

As the number of recipes grows, so does the difficulty of choosing the right one. Especially nowadays, when people are busy and have high obesity rates, they need recipes that are quick and not too high in calories. In this study, I will focus on recipes that can be made in less than two hours with not too high calories and predict if the recipe will take less than one hour based on the information in the recipe.

The dataset is from food.com which contains recipes and reviews. The recipe dataset has important inforamtion like id, number of steps, minutes, tags, and nutrition. It has 83782 rows. The ratings dataset has inforamtion like user ID, recipe ID, and rating and it has 731927 rows.


## Data Cleaning and EDA

### Data cleaning

First, since this study will only focus on recipes that are less than two hours old, I'll be processing the data, removing unneeded columns and adding some columns that make sense.

For the interacts database, I only kept the recipe id and rating because it will eventually be used to calculate the average rating of the recipes.

|   recipe_id |   rating |
|------------:|---------:|
|       40893 |        5 |
|       85009 |        5 |
|       85009 |        5 |
|      120345 |        0 |
|      120345 |        2 |

For the recipes database, first, I converted all the values in the dataset that should be lists, but are actually strings, into lists. then, I added two new columns : the number of tags and the calorie value. Finally, I removed the unneeded columns : "contributor_id", "tags", "nutrition", "steps", "description", "ingredients" (They were removed because they were partly difficult to use for data analysis and partly because I had already extracted useful information).Then I calculated the average score for each recipe and added it to the recipes dataset. Finally, I filtered out recipes that took more than two hours to make, as well as recipes with more than 1,500 calories. Finally, I split the minutes into categorical features, namely "0-59" and "60-120".

| name                                 |     id |   minutes | submitted   |   n_steps |   n_ingredients |   n_tags |   calories |   avg_rating |
|:-------------------------------------|-------:|----------:|:------------|----------:|----------------:|---------:|-----------:|-------------:|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27  |        10 |               9 |       14 |      138.4 |            4 |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11  |        12 |              11 |        9 |      595.1 |            5 |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30  |         6 |               9 |       10 |      194.8 |            5 |
| millionaire pound cake               | 286009 |       120 | 2008-02-12  |         7 |               7 |       20 |      878.3 |            5 |
| 2000 meatloaf                        | 475785 |        90 | 2012-03-06  |        17 |              13 |       10 |      267   |            5 |

### EDA

For EDA, first, I checked the distribution of all the features, and one interesting finding was that the avg_rating feature is heavily skewed, with an extremely large number of recipes rated 5.0, so skewed that I don't think it provides any useful information for prediction purposes anymore. The skew is so bad that I don't think it provides any useful information when doing prediction. Therefore it will not be used in the prediction. And for the other features, although they are also skewed, we can make their distribution normal by taking log.

<iframe src="assets/uni_avg_rating.html" width=800 height=600 frameBorder=0></iframe>

Then, for the bivariate analysis, I analyzed the relationship between "n_steps", "n_ingredients", and "calories". None of the three figures (see below for an example of the relationship between steps and ingredients) show a strong correlation. This is what we would expect, as we do not want to be influenced by multicollinearity when we make our predictions.

<iframe src="assets/bi_step_ing.html" width=800 height=600 frameBorder=0></iframe>

Finally I did aggregation on feature "tags" and calculated its max, mean, and median for different time classifications, and found it to be almost identical respectively. This likely indicates that it provides little meaningful information for prediction.

<iframe src="assets/agg_tag_min.html" width=800 height=600 frameBorder=0></iframe>

## Assessment of Missingness

### NMAR Analysis

(For this part, I will use the unfiltered dataset). We can see that this dataset has two features containing NA values. One is Name and the other is avg_rating. I think that for Name, the NA value generated is most likely **NMAR**, probably because: some of the text is from a foreign language and can't be decoded with the current decoder. To prevent the program from crashing, the names are changed to NaN.

table

### Missingness Dependency

And for avg_rating, I think it's likely to be MAR because a late submitted recipes will get fewer reviews and ratings compared to early submitted recipes, and therefore more likely to get NA value. We split the database into two parts, one where "avg_rating" is missing and the other where it is not, and we can see that the distribution of ids is not similar. We can see that for recipes where avg_rating is not NA, they have more ids that are small (i.e. submitted early).

<iframe src="assets/mis_id_rate.html" width=800 height=600 frameBorder=0></iframe>

I then performed a permutation test to randomly permute the ids of the recipes and obtained the distribution of the K-S Statistic. as can be seen, the p_value of the values that we observe for the K-S Statistic is essentially 0.

<iframe src="assets/fig_ks_id.html" width=800 height=600 frameBorder=0></iframe>

## Hypothesis Testing

Now, I'm going to perform a Hypothesis test. The Null hypothesis is: Recipes that take less time (<1 h) and those that take more time (>= 1 h) have the same steps. Alternative hypothesis is: Recipes that take less time (<1 h) have higher steps than those that take longer (>= 1 h). I will perform permutation test. test statistic is difference in mean of n_steps. significance level is 0.05.

<iframe src="assets/fig_step4.html" width=800 height=600 frameBorder=0></iframe>

As can be seen, it is almost impossible to see this distribution simply because of randomness. p-value is almost 0. The significance of understanding this is that we can assume that the feature n_steps can be of great help when doing prediction. This is because it has a strong correlation with the time spent on the recipe.

## Framing a Prediction Problem

