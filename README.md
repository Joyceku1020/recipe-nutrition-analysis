# Nutritional Insights: Predicting Calories in Recipes

Joyce Ku

joyceku@umich.edu

## Introduction
---
This project explores nutritional patterns in recipes sourced from [food.com](https://www.food.com), with a focus on predicting calorie content and understanding how nutritional profiles vary across different cuisines. 

With the rising interest in health-conscious eating, a relevant question that may be asked is: **How do factors such as type of cuisine, type of ingredients, nutritional information, and other relevant tags result in a higher calorie recipe?** This question matters because it can help consumers make more informed dietary choices, understand cultural differences in food composition, and potentially support healthier cooking decisions.

To answer this, I used a subset of a raw dataset on the site posted after 2008 containing `recipes` and `interactions`. 

The `recipes` dataset has **83,782 rows** with one row for each recipe and details out each recipe's information. The following columns were used in the data analysis:

| `id` | Recipe ID |
| `tags` | A list of keywords including possible cuisine labels (e.g., 'american', 'french') |
| `nutrition` | A list of nutrition information in the format [calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates]  |
| `ingredients` | A list of ingredients used in the recipe |

The `interactions` dataset has **731,927 rows** with one row for each rating for one of the recipes mentioned in the `recipes` dataset. The following columns were used in the data analysis:

| `recipe_id` | Recipe ID |
| `rating` | Recipe rating from 1-5 or not rated (NaN) |

Through this project, I aim to analyze variations in nutritional value in recipes based on types of cuisines and types of ingredients. Utimately, the goal is to predict a recipe's calories based on its nutritional breakdown. 



## Data Cleaning and Exploratory Data Analysis
---

### Data Cleaning

I took several measures to clean and label my data:

1. I left merged the `recipes` dataset with the `interactions` dataset on the recipe ID columns. After merging, the resulting dataset had 234,428 rows. 

2. In the merged dataset, I filled all ratings of 0 with np.nan. The reasoning behind this is because a rating that is equal to 0 would indicate a missing rating where the user hasn't rated the recipe at all. If I were to leave the ratings that are equal to 0 as is, this would lead to misleading analysis since it would decrease the average ratings of each recipe. Therefore, replacing 0 with np.nan prevents skewing averages and ensures that the model treats them as unrated, not negative feedback.

3. Then, I averaged all of the ratings for each recipe and I added it as a column called `avg_rating` to the DataFrame. This was done by grouping the merged dataset by recipe_id and calculating the mean of the rating column. The resulting average ratings were then merged back into the main dataset. This ensures that each recipe has a single, representative rating value, which can be useful for later analysis or filtering.

4. The nutrition column originally contained a single string with all nutritional information in a list format. To make this data usable, I split the string into separate numerical columns for each nutrient. These columns included: 
    - calories
    - total fat
    - sugar
    - sodium
    - protein
    - saturated fat
    - carbohydrates

5. After analyzing the data, it was found that there were extreme outliers in the dataset that would affect the overall visualization and interpretation of the results. Specifically in the calories column, the dataset contained recipes with over 10,000 calories, which made it harder to identify meaningful trends. To focus the analysis on more realistic, single-serving recipes, only recipes with fewer than 2,000 calories were included.

6. I grouped the resulting dataset by recipe ID again to ensure there was only one row per recipe, since the focus of the analysis is not on individual reviews.

Below is a preview of the cleaned dataset sorted by average rating from high to low:

|   recipe_id | name                                |   minutes | tags                                                                                                                                                                                                                                                                                                                   |   n_steps | ingredients                                                                                                                                                                                            |   n_ingredients |   calories |   total_fat |   sugar |   sodium |   protein |   saturated_fat |   carbohydrates |   avg_rating |
|------------:|:------------------------------------|----------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------:|-----------:|------------:|--------:|---------:|----------:|----------------:|----------------:|-------------:|
|      373693 | savory palmiers                     |        75 | ['time-to-make', 'course', 'main-ingredient', 'cuisine', 'preparation', 'occasion', 'appetizers', 'eggs-dairy', 'french', 'european', 'dinner-party', 'cheese', 'taste-mood', 'savory', '4-hours-or-less']                                                                                                             |        14 | ['puff pastry', 'pesto sauce', 'goat cheese', 'sun-dried tomato packed in oil', 'pine nuts', 'kosher salt']                                                                                            |               6 |       69.7 |           7 |       0 |        1 |         1 |               5 |               1 |            5 |
|      396958 | sassy chicken salad                 |         5 | ['15-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'salads', 'eggs-dairy', 'fruit', 'poultry', 'easy', 'nuts', 'chicken', 'dietary', 'low-sodium', 'low-cholesterol', 'healthy-2', 'low-in-something', 'meat', 'chicken-breasts', '3-steps-or-less']                                   |         3 | ['tomatoes', 'mayonnaise', 'honey', 'chicken breast', 'celery', 'seedless grapes', 'pecans']                                                                                                           |               7 |      330   |          35 |      59 |       10 |        20 |              16 |               7 |            5 |
|      396913 | nutty roasted garlic ricotta spread |        10 | ['15-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'cuisine', 'preparation', '5-ingredients-or-less', 'appetizers', 'eggs-dairy', 'easy', 'european', 'vegetarian', 'italian', 'spreads', 'cheese', 'dietary', 'low-sodium', 'low-carb', 'low-in-something', 'presentation', 'served-cold']           |         4 | ['pine nuts', 'roasted garlic', 'ricotta cheese', 'salt and pepper']                                                                                                                                   |               4 |      164.3 |          19 |       2 |        1 |        15 |              25 |               1 |            5 |
|      396919 | fruit and nut chocolate chunks      |        35 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'cuisine', 'preparation', 'occasion', 'north-american', 'desserts', 'fruit', 'holiday-event', 'candy', 'chocolate', 'nuts', 'dietary', 'christmas', 'low-sodium', 'low-cholesterol', 'halloween', 'valentines-day', 'low-in-something', 'berries'] |         9 | ['bittersweet chocolate', 'vegetable oil', 'dried cranberries', 'raisins', 'shelled pistachios', 'roasted cashews']                                                                                    |               6 |       36.4 |           3 |       7 |        1 |         1 |               1 |               1 |            5 |
|      396922 | pumpkin pie fudge                   |        30 | ['30-minutes-or-less', 'time-to-make', 'course', 'preparation', 'occasion', 'for-large-groups', 'fudge', 'desserts', 'fall', 'holiday-event', 'candy', 'seasonal', 'number-of-servings']                                                                                                                               |        12 | ['granulated sugar', 'light brown sugar', 'butter', 'evaporated milk', 'pumpkin puree', 'pumpkin pie spice', 'white chocolate chips', 'cinnamon baking chips', 'marshmallow creme', 'vanilla extract'] |              10 |       59.5 |           3 |      34 |        0 |         0 |               7 |               3 |            5 |


### Univariate Analysis

Since there were extreme outliers in the calories column, I decided to only analyze recipes with calories less than 2000.

I first examined the distribution of calories across all recipes. The histogram below shows how calories are distributed:

<iframe
 src="assets/calorie-distribution.html"
 width="800"
 height="400"
 frameborder="0"
 ></iframe>

The plot reveals that most recipes fall under 1,000 calories, with a noticeable peak between 100 to 300 calories. The distribution is right-skewed, eamning there are more low-calorie recipes and fewer high-calorie ones, even after removing the extreme outliers. 



I also explored the most common cuisine tags in the recipes, excluding any overlapping categories. To keep the analysis more meaningful, I removed broad labels like "European," since they often contain more specific regional cuisines. On the other hand, I chose to keep "American" as a single category, combining its regional variations due to their overall similarity. The bar chart below shows the number of recipies per cuisine:


<iframe
 src="assets/recipes-per-cuisine.html"
 width="1000"
 height="400"
 frameborder="0"
 ></iframe>

The bar chart shows American cuisine is by far the most common, with over 9,000 recipes which is more than three times the next most frequent cuisines. Italian and Mexican cuisines follow, with around 2,500 and 2,100 recipes respectively. Beyond these top three, the remaining cuisines appear at much lower but relatively similar frequencies, suggesting that while the dataset is diverse, it has a strong bias toward American-style recipes.





### Bivariate Analysis


### Interesting Aggregates


### Imputation




## Framing a Prediction Problem
---

## Baseline Model
---

## Final Model
---