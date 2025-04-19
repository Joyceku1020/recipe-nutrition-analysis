# Nutritional Insights: Predicting Calories in Recipes

Joyce Ku | joyceku@umich.edu

## Introduction
---
This project explores nutritional patterns in recipes sourced from [food.com](https://www.food.com), with a focus on predicting calorie content and understanding how nutritional profiles vary across different cuisines. With the rising interest in health-conscious eating, a relevant question that may be asked is **_How do factors like type of cuisine, type of ingredients, nutritional information, and other relevant tags result in a higher calorie recipe?_** This question matters because it can help consumers make more informed dietary choices, understand cultural differences in food composition, and potentially support healthier cooking decisions.

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

After merging both the recipes and reviews datasets on the recipe ID, the resulting dataset had 234,428 rows. 

## Framing a Prediction Problem
---

## Baseline Model
---

## Final Model
---