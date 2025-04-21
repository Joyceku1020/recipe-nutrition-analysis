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

Below is a preview of the cleaned dataset:

|   recipe_id | name                                  |   minutes | tags                                                                                                                                                                                                                                                                                                                                    | nutrition                                                        |   n_steps | steps                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | ingredients                                                                                                                            |   n_ingredients |   calories |   total_fat |   sugar |   sodium |   protein |   saturated_fat |   carbohydrates |   avg_rating |
|------------:|:--------------------------------------|----------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------|----------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------|----------------:|-----------:|------------:|--------:|---------:|----------:|----------------:|----------------:|-------------:|
|      275022 | impossible macaroni and cheese pie    |        50 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'main-dish', 'eggs-dairy', 'pasta', 'easy', 'cheese', 'dietary', 'high-calcium', 'high-in-something', 'pasta-rice-and-grains', 'elbow-macaroni']                                                                                                     | ['386.1', ' 34.0', ' 7.0', ' 24.0', ' 41.0', ' 62.0', ' 8.0']    |        11 | ['heat oven to 400 degrees fahrenheit', 'grease a pie plate 10 x 1 1 / 2 inches', 'mix 2 cups cheese and the macaroni', 'sprinkle mixture in the plate', 'beat remaining ingredients , except the 1 / 4 cup cheese , until smooth , 15 seconds in a blender on high , or 1 minute with a hand beater', 'pour into plate', 'bake until knife inserted in center comes out clean - about 40 minutes', 'sprinkle with 1 / 4 cup cheese', 'bake until cheese is melted , 1 to 2 minutes', 'cool 10 minutes', 'serves 6 to 8 people']                                                     | ['cheddar cheese', 'macaroni', 'milk', 'eggs', 'bisquick', 'salt', 'red pepper sauce']                                                 |               7 |      386.1 |          34 |       7 |       24 |        41 |              62 |               8 |            3 |
|      275024 | impossible rhubarb pie                |        55 | ['60-minutes-or-less', 'time-to-make', 'course', 'preparation', 'healthy', 'pies-and-tarts', 'desserts', 'pies', 'dietary']                                                                                                                                                                                                             | ['377.1', ' 18.0', ' 208.0', ' 13.0', ' 13.0', ' 30.0', ' 20.0'] |         6 | ['heat oven to 375 degrees', 'grease 10" pan , put rhubarb in pan', 'blend all remaining ingredients for 3 minutes', 'pour over rhubarb', 'let set for a few minutes', 'bake 40 to 45 minutes']                                                                                                                                                                                                                                                                                                                                                                                      | ['rhubarb', 'eggs', 'bisquick', 'butter', 'salt', 'sugar', 'vanilla', 'milk']                                                          |               8 |      377.1 |          18 |     208 |       13 |        13 |              30 |              20 |            3 |
|      275026 | impossible seafood pie                |        45 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'very-low-carbs', 'main-dish', 'eggs-dairy', 'seafood', 'crab', 'cheese', 'dietary', 'low-sodium', 'low-calorie', 'low-carb', 'low-in-something', 'shellfish']                                                                                       | ['326.6', ' 30.0', ' 12.0', ' 27.0', ' 37.0', ' 51.0', ' 5.0']   |         7 | ['preheat oven to 400f', 'lightly grease large pie plate', 'mix crabmeat , cheeses and onion in pie plate', 'mix remaining ingredients in blender until smooth', 'slowly pour liquid mixture into pie plate', 'bake until golden brown for 35 to 40 minutes', 'let stand 5 minutes before cutting']                                                                                                                                                                                                                                                                                  | ['frozen crabmeat', 'sharp cheddar cheese', 'cream cheese', 'onion', 'milk', 'bisquick', 'eggs', 'salt', 'nutmeg']                     |               9 |      326.6 |          30 |      12 |       27 |        37 |              51 |               5 |            3 |
|      275030 | paula deen s caramel apple cheesecake |        45 | ['60-minutes-or-less', 'time-to-make', 'course', 'preparation', 'occasion', 'desserts', 'cheesecake', 'gifts', 'taste-mood', 'sweet']                                                                                                                                                                                                   | ['577.7', ' 53.0', ' 149.0', ' 19.0', ' 14.0', ' 67.0', ' 21.0'] |        11 | ['preheat oven to 350f', 'reserve 3 / 4 cup apple filling , and set aside', 'spoon remaining apple filling into the crust', 'beat together in large bowl , cream cheese , sugar , vanilla , eggs', 'pour over pie filling', 'bake for 35 minutes', 'cool', 'meanwhile , mix reserved pie filling and caramel topping and heat for 1 minute in a small saucepan', 'spread warm topping evenly over cooked , cooled cheesecake', 'decorate entire edge of cake with the 12 pecan halves , and sprinkle middle of cheesecake with chopped pecans', 'refrigerate , share , and enjoy !'] | ['apple pie filling', 'graham cracker crust', 'cream cheese', 'sugar', 'vanilla', 'eggs', 'caramel topping', 'pecan halves', 'pecans'] |               9 |      577.7 |          53 |     149 |       19 |        14 |              67 |              21 |            5 |
|      275032 | midori poached pears                  |        25 | ['lactose', '30-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'cuisine', 'preparation', 'occasion', 'south-west-pacific', 'desserts', 'fruit', 'australian', 'easy', 'beginner-cook', 'dinner-party', 'summer', 'dietary', 'gluten-free', 'seasonal', 'egg-free', 'free-of-something', 'pears', 'taste-mood', 'sweet'] | ['386.9', ' 0.0', ' 347.0', ' 0.0', ' 1.0', ' 0.0', ' 33.0']     |         8 | ['bring midori , sugar , spices , rinds and water to the boil', 'simmer for 5 minutes', 'peel the pears and remove the base end but leave the stem intact', 'place pears in hot liquid and simmer for approximately 15mins or until cooked', 'cooking time depends on how ripe the pears are', 'place each pear on a dessert plate', 'top each pear with 2 tablespoons reserved poaching liquid', 'garnish with orange rind curls and mint sprigs']                                                                                                                                  | ['midori melon liqueur', 'water', 'caster sugar', 'cinnamon stick', 'vanilla pod', 'lemon rind', 'orange rind', 'pear', 'mint']        |               9 |      386.9 |           0 |     347 |        0 |         1 |               0 |              33 |            5 |



### Univariate Analysis

Since there were extreme outliers in the calories column, I decided to only analyze recipes with calories less than 2000.

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