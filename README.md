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

Below is a preview of the cleaned dataset:



### Univariate Analysis


### Bivariate Analysis


### Interesting Aggregates


### Imputation




## Framing a Prediction Problem
---

## Baseline Model
---

## Final Model
---