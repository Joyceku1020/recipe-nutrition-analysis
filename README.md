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

|   recipe_id | name                                                |   minutes | tags                                                                                                                                                                                                                                                                    | nutrition                                                    |   n_steps | ingredients                                                                                                                                                                                                                           |   n_ingredients |   calories |   total_fat |   sugar |   sodium |   protein |   saturated_fat |   carbohydrates |   avg_rating |
|------------:|:----------------------------------------------------|----------:|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------|----------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------:|-----------:|------------:|--------:|---------:|----------:|----------------:|----------------:|-------------:|
|      422780 | sneaky snack bars                                   |       140 | ['time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'healthy', 'lunch', 'kid-friendly', 'grains', 'dietary', 'pasta-rice-and-grains', 'number-of-servings', '4-hours-or-less']                                                             | ['120.7', ' 4.0', ' 50.0', ' 4.0', ' 3.0', ' 8.0', ' 8.0']   |         9 | ['cheerios toasted oat cereal', 'bran flakes', 'dried cranberries', 'dried apricot', 'coconut flakes', 'butter', 'marshmallows']                                                                                                      |               7 |      120.7 |           4 |      50 |        4 |         3 |               8 |               8 |            5 |
|      284673 | blueberry lemon muffins   with yellow squash        |        26 | ['30-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'pancakes-and-waffles', 'breakfast', 'vegetables', 'easy', 'squash']                                                                                                                 | ['160.5', ' 2.0', ' 56.0', ' 10.0', ' 8.0', ' 2.0', ' 10.0'] |        11 | ['nonstick cooking spray', 'brown sugar', 'trans-fat free margarine', 'lemon low fat yogurt', 'blueberries', 'squash puree', 'egg', 'lemon extract', 'lemon zest', 'flour', 'flax seed meal', 'baking powder', 'baking soda', 'salt'] |              14 |      160.5 |           2 |      56 |       10 |         8 |               2 |              10 |            5 |
|      481999 | cranberry and orange iced tea                       |        15 | ['15-minutes-or-less', 'time-to-make', 'course', 'cuisine', 'preparation', '5-ingredients-or-less', 'beverages', 'australian', 'easy', 'non-alcoholic']                                                                                                                 | ['89.0', ' 0.0', ' 79.0', ' 0.0', ' 0.0', ' 0.0', ' 7.0']    |        17 | ['orange', 'tea bags', 'boiling water', 'cranberry juice', 'caster sugar']                                                                                                                                                            |               5 |       89   |           0 |      79 |        0 |         0 |               0 |               7 |            5 |
|      481998 | desire potatoes pan cooked with rosemary and garlic |        35 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'cuisine', 'preparation', 'low-protein', 'side-dishes', 'potatoes', 'vegetables', 'australian', 'vegan', 'vegetarian', 'dietary', 'low-sodium', 'low-cholesterol', 'healthy-2', 'low-in-something'] | ['304.6', ' 28.0', ' 5.0', ' 0.0', ' 8.0', ' 12.0', ' 10.0'] |        13 | ['potatoes', 'sea salt', 'virgin olive oil', 'garlic cloves', 'fresh rosemary']                                                                                                                                                       |               5 |      304.6 |          28 |       5 |        0 |         8 |              12 |              10 |            5 |
|      284605 | brown rice salad with grilled chicken               |        20 | ['30-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'main-dish', 'salads', 'poultry', 'chicken', 'one-dish-meal', 'meat']                                                                                                                | ['195.1', ' 10.0', ' 4.0', ' 13.0', ' 25.0', ' 5.0', ' 6.0'] |         4 | ['cooked brown rice', 'grilled chicken breasts', 'sweet red pepper', 'celery ribs', 'green onion', 'fresh parsley', 'cider vinegar', 'olive oil', 'lemon juice', 'salt', 'pepper']                                                    |              11 |      195.1 |          10 |       4 |       13 |        25 |               5 |               6 |            5 |



### Univariate Analysis

Since there were extreme outliers in the calories column, I decided to only analyze recipes with calories less than 2000.

I first examined the distribution of calories across all recipes. The histogram below shows how calories are distributed:

<iframe
 src="assets/calories-distribution.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

The plot reveals that most recipes fall under 1,000 calories, with a noticeable peak between 100 to 300 calories. The distribution is right-skewed, eamning there are more low-calorie recipes and fewer high-calorie ones, even after removing the extreme outliers. 



I displayed most common cuisine tags in recipes, without including overlapping cuisines. I removed the more general cuisine varieties. For example, I removed american because of the american subsection cuisines.  

### Bivariate Analysis


### Interesting Aggregates


### Imputation




## Framing a Prediction Problem
---

## Baseline Model
---

## Final Model
---