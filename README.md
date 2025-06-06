# Steps to Culinary Perfection

by Chloe Suwignjo & Irina Vardapetyan

# Introduction

With the advent of health influences and the surge of healthy living trends, dietary breakdowns (in terms of calories, protein content, etc.) has become exponentially more prevalent in our society and social media. However, many individuals stray away from healthy living because of time constraints and busy lifestyle practices. Many resort to unhealthier options because they simply "do not have time" or because the "recipes are too complicated". In order to break down whether this observation amongst busy people is true, we decided to investigate the length of a recipe from its **number of steps**.

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

After all cleaning, in total, the cleaned dataset contains 73667 rows/recipes.

## Univariate Analysis

To better understand our dataset and identify interesting observations, we perform exploratory data analysis. This plot shows the histogram of the distribution of the number of steps.

<iframe
  src="assets/univariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram above displays a right-skew. The bulk of the recipes range from about 100–600 calories, with a long tail up to around 2000, which was our cutoff. The right tail indicates that there are a few recipes with extremely high calorie counts, but the bulk of dishes cluster under 600 calories.

## Bivariate Analysis

We also conduct bivariate analysis to understand the relationship between different columns in our dataset. This plot shows the distribution of calories for recipes with varying number of steps.

<iframe
  src="assets/bivariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The boxplots above illustrate how, as the number of steps in a recipe increase, median calories generally drift upward. Bins [8,13) and [13,18) have the largest medians of around 350-400 calories. With recipes with more than 20 steps, the median calorie level begins to stabilize but with wider interquartile ranges, implying that there is more variability. 