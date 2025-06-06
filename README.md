# Steps to Culinary Perfection

by Chloe Suwignjo & Irina Vardapetyan

# Introduction

With the advent of health influences and the surge of healthy living trends, dietary breakdowns (in terms of calories, protein content, etc.) has become exponentially more prevalent in our society and social media. However, many individuals stray away from healthy living because of time constraints and busy lifestyle practices. Many resort to unhealthier options because they simply "do not have time" or because the "recipes are too complicated". In order to break down whether this observation amongst busy people is true, we decided to investigate the length of a recipe from its **number of steps**. In this project, we ask the question: **do“simpler” recipes (as measured by number of steps or ingredients) actually correspond to dishes with a lower calorie count?**

To explore this, we use a dataset of recipes and ratings from [food.com](https://food.com), which was originally scraped and obtained by Majumder, et al for a research paper titled [Generating Personalized Recipes from Historical User Preferences](https://cseweb.ucsd.edu/~jmcauley/pdfs/emnlp19c.pdf). 

The first dataset, `recipes`, contains 83,782 rows of recipes and 12 columns as follows:

|  **Column**       |  **Description** |
| --------------    |  -----------     |
|`name`	            |   Recipe name    |
|`id`               |   Recipe ID      |
|`minutes`          |   Minutes to prepare recipe   |
|`contributor_id`   |	User ID who submitted this recipe   |
|`submitted`        |	Date recipe was submitted   |
|`tags`	            |   Food.com tags for recipe    |
|`nutrition`        |	Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value”   |
|`n_steps`          |	Number of steps in recipe   |
|`steps`            |	Text for recipe steps, in order |
|`description`      |	User-provided description   |

The second dataset, `interactions`, contains 731,927 rows of reviews and 5 columns as follows:

|**Column**  |	**Description**     |
| ---------  | -----------------    |
|`user_id `  |	User ID             |
|`recipe_id` |	Recipe ID           |
|`date`	     |  Date of interaction |
|`rating`    |	Rating given        |
|`review`    |	Review text         |

---

# Data Cleaning and Exploratory Data Analysis

## Data Cleaning

In order to perform meaningful and relevant analysis, the first step of this project is to clean the datasets. We performed the following steps to prepare the dataset for analysis:
1. Left merge the `recipes` and `interactions` dataframes so each review is associated with its recipe.
2. Fill in all ratings of 0 in the `rating` column with `np.nan`. This is done because 1 is the lowest possible rating in our grading scale.
3. Groupby `id` and aggregate using `mean()` to obtain the average rating for each recipe, and merge this newly created series with our dataframe in a column called `average_rating`.
4. Since the `review` column does not have much use for this project, we drop it. Additionally, because after merging the average ratings, some recipes have multiple rows (e.g. impossible macaroni and cheese pie as displayed above). However, we only want to keep one row for each recipe. Hence, we drop recipes with multiple rows in the dataframe.
5. Create two new columns called `calories` and `protein` from the `nutrition` column.

Since we discovered that there are many outliers in the dataset, we decided to only keep recipes that meet these conditions:
- More than 0 and less than 2000 calories, as 2000 calories is the average daily caloric intake for an adult.
- More than 3 ingredients.
- More than 3 and less than 40 steps.

For the purpose of our analysis, we define a recipe has a high number of steps if it has 9 or more steps, based on the median for `n_steps`. Upon cleaning, here are the first 5 rows of the cleaned dataset. For display conciseness, only columns that will be used for our analysis is displayed.

| name                                  |     id |   minutes | nutrition                                    |   n_steps | description                                                                                                                                                                                                                      | ingredients                                                                                                                            |   n_ingredients |   average_rating |   calories |   protein | high_n_steps   |
|:--------------------------------------|-------:|----------:|:---------------------------------------------|----------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------|----------------:|-----------------:|-----------:|----------:|:---------------|
| impossible macaroni and cheese pie    | 275022 |        50 | [386.1, 34.0, 7.0, 24.0, 41.0, 62.0, 8.0]    |        11 | one of my mom's favorite bisquick recipes. this brings back memories!                                                                                                                                                            | ['cheddar cheese', 'macaroni', 'milk', 'eggs', 'bisquick', 'salt', 'red pepper sauce']                                                 |               7 |                3 |      386.1 |        41 | True           |
| impossible rhubarb pie                | 275024 |        55 | [377.1, 18.0, 208.0, 13.0, 13.0, 30.0, 20.0] |         6 | a childhood favorite of mine. my mom loved it because it cut down on how much time to make it.                                                                                                                                   | ['rhubarb', 'eggs', 'bisquick', 'butter', 'salt', 'sugar', 'vanilla', 'milk']                                                          |               8 |                3 |      377.1 |        13 | False          |
| impossible seafood pie                | 275026 |        45 | [326.6, 30.0, 12.0, 27.0, 37.0, 51.0, 5.0]   |         7 | this is an oldie but a goodie. mom's stand by for company. good enough for us on a special occasion or if company came over!                                                                                                     | ['frozen crabmeat', 'sharp cheddar cheese', 'cream cheese', 'onion', 'milk', 'bisquick', 'eggs', 'salt', 'nutmeg']                     |               9 |                3 |      326.6 |        37 | False          |
| paula deen s caramel apple cheesecake | 275030 |        45 | [577.7, 53.0, 149.0, 19.0, 14.0, 67.0, 21.0] |        11 | thank you paula deen!  hubby just happened to be watching with me one day when she made these and it will always be requested in our home!  it's very easy to make and such a fun twist on a plain cheesecake.  it's a must try! | ['apple pie filling', 'graham cracker crust', 'cream cheese', 'sugar', 'vanilla', 'eggs', 'caramel topping', 'pecan halves', 'pecans'] |               9 |                5 |      577.7 |        14 | True           |
| midori poached pears                  | 275032 |        25 | [386.9, 0.0, 347.0, 0.0, 1.0, 0.0, 33.0]     |         8 | the green colour looks fabulous and the taste is heavenly. serve with a raspberry coulis. keep enough rind of the orange and lemon for garnish.                                                                                  | ['midori melon liqueur', 'water', 'caster sugar', 'cinnamon stick', 'vanilla pod', 'lemon rind', 'orange rind', 'pear', 'mint']        |               9 |                5 |      386.9 |         1 | False          |

After all cleaning, in total, the cleaned dataset contains 73667 rows/recipes and 20 columns.

## Univariate Analysis

To better understand our dataset and identify interesting observations, we perform exploratory data analysis. This plot shows the histogram of the distribution of the number of steps.

<iframe
  src="assets/univariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram above shows us that the number of steps is roughly centered around 8-12, with a slight right skew. There are very few recipes with more than 20 steps and the majority of recipes have approximately 3-15 steps.

## Bivariate Analysis

We also conduct bivariate analysis to understand the relationship between different columns in our dataset. This plot shows the distribution of calories for recipes with varying number of steps.

<iframe
  src="assets/bivariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The boxplots above illustrate how, as the number of steps in a recipe increase, median calories generally drift upward. Bins [8,13) and [13,18) have the largest medians of around 350-400 calories. With recipes with more than 20 steps, the median calorie level begins to stabilize but with wider interquartile ranges, implying that there is more variability. 

## Interesting Aggregates

Here, we grouped by number of ingredients and calculated the mean and median for calories as well as protein for each group. 

|   n_ingredients |   calories_mean |   calories_median |   protein_mean |   protein_median |
|----------------:|----------------:|------------------:|---------------:|-----------------:|
|               3 |         255.048 |             168.8 |        14.6598 |                5 |
|               4 |         289.662 |             199.2 |        17.93   |                8 |
|               5 |         298.096 |             221.8 |        20.1913 |                9 |
|               6 |         319.216 |             245.4 |        22.8654 |               11 |
|               7 |         339.026 |             268.6 |        26.5615 |               14 |
| ... | ... |
|              29 |         722.771 |            607.4  |        69.1429 |               58 |
|              30 |         700.333 |            659.9  |        67.3333 |               67 |
|              31 |         789.5   |            552.8  |        49.5714 |               38 |
|              32 |         697.35  |            697.35 |        66      |               66 |
|              33 |         338.2   |            338.2  |         8      |                8 |

For us to more easily identify the pattern from the resulting table, we plot the results as an overlaid line plot.

<iframe
  src="assets/interesting.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above shows us that as the number of ingredients increases, the mean calories in a recipe gradually rises. Median calories also increase, but less smoothly. Protein shows a similar positive trend: as the number of ingredients rise, so does the mean protein. This highlights a clear nutritional progression. Recipes with fewer ingredients tend to be lower in calories and protein, while more complex recipes with more ingredients both contain more calories and protein. These insights could be useful for those in finding recipes that fulfill their nutritional goals while ensuring they have time to cook the recipe.

---

# Assessment of Missingness

## NMAR Analysis

In this section, we will determine the missingness of the `description` column is. If a column is not missing at random (NMAR), the chance that a value is missing depends on the actual missing value. If a column is missing completely at random (MCAR), the chance that a value is missing is completely independent of other columns and the actual missing value. The missingness of the `description` column is not missing at random (NMAR). In this context, the `description` column can be described as NMAR because of the simplicity or familiarity that is associated with that recipe. For example, certain simple and straightforward recipes such as scrambled or eggs or boiled pasta are so well known that they do not require a description. For these recipes, the contributors trust that the readers are already familiar with the meal so they omit the description.

## Missingness Dependency

To test the missingness of the `rating_missing` column, we perform two permutation tests. First, we investigate if the missingness of `rating_missing` depends on a recipe's number of ingredients (`n_ingredients`).

**Null hypothesis:** The missingness of ratings does not depend on the recipe's number of ingredients.

**Alternative hypothesis:** The missingness of ratings does depend on the recipe's number of ingredients.

**Test statistic:** Total variation distance (TVD)

**Significance level:** 0.05

Now, we plot these distributions to see how rated and unrated recipes differ by ingredient count.

<iframe
  src="assets/ingredients-missingness1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Not missing (the blue curve) tends to be higher for recipes with 7–9 ingredients, which means that rated recipes concentrate around mid-complexity. Missing (the red curve) overtakes after about 10–11 ingredients, showing that recipes with more recipes are more likely to be unrated. Because of these blatant differences, missingness in `average_rating` is MAR with respect to `n_ingredients` rather than MCAR. Since the two lines are close together across all ingredient counts, missingness might be MCAR.  

<iframe
  src="assets/ingredients-missingness1-tvd.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram above shows the distribution of TVDs, and the vertical red line marks our observed TVD. We obtained a p-value of **0.296**, which is the fraction of shuffled TVDs that are greater than or equal to our observed TVD.  

Because the p-value > 0.05, we **fail to reject** the null hypothesis. Thus, the missingness of `average_rating` does not depend on n_ingredients. This implies that it is MCAR.

Next, we will repeat the same permutation test process, but now using `n_steps`. 

**Null hypothesis:** The missingness of ratings does not depend on the recipe's number of steps.

**Alternative hypothesis:** The missingness of ratings does depend on the recipe's number of steps.

**Test statistic:** Total variation distance (TVD)

**Significance level:** 0.05

We plot the distribution of missingness of rating by number of steps.

<iframe
  src="assets/steps-missingness2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

In the graph above, we see that not missing (the blue curve) peaks more strongly at approximately 8 steps, indicating many rated recipes are of moderate complexity. On the other hand, missing (the red curve) is relatively lower in the 3–7 step range and relatively higher in the 14+ step range, showing that extremely simple recipes tend to be rated while extremely long recipes tend not to be rated. Therefore, missingness of average_rating clearly depends on n_steps, confirming that ratings are not MCAR by step count.

<iframe
  src="assets/steps-missingness2-tvd.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram above shows the the null distribution of TVDs if missingness were MCAR with respect to `n_steps`. The red line is our observed TVD. Since `p_value2` is approximately 0.0, which is less than 0.05, we **reject** the null. The missingness of `average_rating` **does depend** on `n_steps`. Hence, the fact that a recipe has “no rating” is not completely random. A plausible explanation for `average_rating` depending on `n_steps` is that a simple recipe with very few steps is likely unrated because it is too well-known. 

---

# Hypothesis Testing

As mentioned in the introduction, we want to explore if recipes has more calories if it has more steps. It could be the case that longer recipes contain more steps, where users might utilize more ingredients and process them more. We conduct a permutation test with the following hypotheses:

**Null hypothesis:** There is no difference in the amount of calories between recipes with low number of steps (less than 9) and recipes with high number of steps (greater than or equal to 9).

**Alternative hypothesis:** The amount of calories for recipes with high number of steps is higher than recipes with low number of steps.

**Test statistic:** Signed difference in means

**Significance level:** 0.05

We shuffled the `high_n_steps` column (which we created in the data cleaning process) 500 times and computed the signed difference in means. The distribution of the aforementioned can be seen in the plot below.

<iframe
  src="assets/hypo-test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram above shows the differences in mean calories if the `high_n_steps` label were not related to calorie count. Our observed difference is 90.62, which gives us a p-value of **0.0**.

Since the p-value is less than 0.05, we **reject** the null hypothesis and conclude that recipes with higher step counts differ in mean calories from recipes with lower step counts. This is not surprising, as we expect recipes with higher step counts to be more elaborate and hence, have a higher average amount of calories.

---

# Framing a Prediction Problem

Building upon our hypothesis tests, our objective is to **predict the number of steps** (the response variable) required for a recipe using various features from our dataset like preparation time (`minutes`), the number of ingredients (`n_ingredients`), and ingredient‐presence indicators (for example, sugar, flour, etc.). This prediction problem is a regression problem since our model is predicting a continuous numeric value. 

We use Root Mean Squared Error (RMSE) to measure how far our predictions are from the actual values. RMSE is an appropriate measure because we care about the average magnitude of the prediction error (in terms of steps). This makes it easier for us to directly compare how off our prediction is on average from the actual values. For example, if a recipe has 10 steps, our model will predict the RMSE in the same units as the response variable (number of steps) which is preferable for its ease and clarity when comparing the predicted and actual value. 

---

# Baseline Model

A baseline model serves as a benchmark because it uses only the most straightforward features to predict our target variable. In this project, the variable we are trying to predict is the number of steps (`n_steps`) required to prepare a recipe. By starting with a very basic model, we can determine how accurately just the two numerical attributes can predict 'n_steps'. We chose as the total preparation time (`minutes`) and the number of ingredients (`n_ingredients`) as the features used to predict the number of steps; both continuous features. 

In order to create the baseline model, we will fit a linear regression model that uses `minutes` and `n_ingredients`. Our intuition behind this is that longer recipes will require more steps, and more ingredients usually correlate with more preparation steps. We used `StandardScaler` to standardize the features and a 80% and 20% train/test split before fitting the model.

This baseline model yielded:
- Training R²: 0.222
- Training RMSE: 5.018
- Test R²: 0.225
- Test RMSE: 5.003

Our training R² tells us that ~22.2% of the variance in `n_steps` can be explained by `minutes` and `n_ingredients`. Our training RMSE and test RMSE are quite close; this is a good sign as it shows that we are not overfitting our data. So despite the RMSE of 5, which means that our model's prediction on average is off by 5 steps, it generalizes quite well to unseen data. Based on these results, our model is a good start for a baseline.

---

# Final Model

To go beyond the baseline model, we decided to create binary indicator columns for the presence of common cooking ingredients from our experience as well as a `has_seafood` flag. These extra features can help capture whether certain ingredient categories tend to require more steps. For example, recipes with flour, eggs, and butter might be a baking recipe, which typically takes a longer preparation time and baking time, and recipes with beef might take longer to roast or grill. 

The ingredients that we included are features are beef, chicken, pork, flour, eggs, cheese, butter, pasta, sugar, rice, potato (11 ingredients). We created boolean indicator columns called `has_ingredient`, where each common ingredient is its own column along with `has_seafood` with one hot encoding. Hence, our final models include 14 features.

We experimented on several different modeling algorithms, such as Lasso and Random Forest Regressor. After comparing the R² and RMSE of each model, we decided to use a **Random Forest Regressor** as our final model. Because decision trees are prone to having high bias and variance, we used `GridSearchCV` to tune the best hyperparameters for 
1. `max_depth`: maximum depth of each decision tree
2. `max_features`: number of features to consider when looking for the best split
3. `min_samples_split`: minimum number of samples required to split a node, prevent overfitting
4. `n_estimators`: number of trees in the forest

With this, we were able to build a more robust and accurate predictive model that outperforms our baseline and models with other algorithms. After grid search, our optimal hyperparameters are 10 for `max_depth`, sqrt for `max_features`, 2 for `min_samples_split`, and 150 for `n_estimators`. With these, we obtained the following metrics for the final model:
- Training R²: 0.336
- Training RMSE: 4.633
- Test R²: 0.297
- Test RMSE: 4.76

Compared to the baseline, our test R² improved from 0.225 to 0.297, and test RMSE decreased from 5.003 to 4.76. Hence, the model is on average off by 4.76 steps now. While this is not a significant improvement, this model is still better than the baseline because it uses more features, which more accurately predicts number of steps based on a recipe's ingredients and uses a more powerful predictive algorithm with the random forest. 

---

# Fairness Analysis

Our objective is to check whether the chosen Random Forest Regressor model performs differently on “high‐step” recipes vs. “low‐step” recipes. Recall that we define a recipe has a high number of steps if it contains greater or equal than 9 steps. We measure performance in terms of RMSE and perform a permutation test to see if any observed difference is statistically significant. To do so, we will compute the observed difference, permute the `high_n_steps` labels among the test set, recompute RMSE difference, and build an empirical null distribution, and finally see how often the permuted difference are greater than or equal to the observed difference to get a p‐value.

**Null hypothesis:** Our model is fair and its RMSE is the same for both high-step and low-step recipes. Any differences is due to random chance.

**Alternative hypothesis:** Our model is not fair and its RMSE is the higher for both high-step than low-step recipes.

**Test statistic:** Difference in RMSE

**Significance level:** 0.05

<iframe
  src="assets/fairness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The plot above visualizes what differences in RMSE (high vs. low step) we’d expect if the model had no real difference (`high_n_steps` labeling were random). The red line is our observed difference (how much worse or better the model does on high‐step recipes compared to low‐step). The printed p_value above tells us how unusual the observed difference is. Since we obtain a p-value of **0.0**, less than 0.05, we reject the null hypothesis that there is no difference.

Our model’s performance is statistically different across those two groups, indicating a fairness concern as it performs better for low step than high step recipes. This might be the case because different authors write recipes differently. Some authors might be have concise instructions (i.e. repeat the step), while some others might list down every single step in a very detailed manner. Finally, two recipes with flour and eggs can differ drastically in steps if one involves folding layers and multiple resting times (e.g. breads), or just a simple fry and quick cook on the stove (e.g. pancakes). 