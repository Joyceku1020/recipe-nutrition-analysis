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


I also explored the most common cuisine tags in the recipes, excluding any overlapping categories. I gathered all of the cuisines from the top 300 most used tags. To keep the analysis more meaningful, I removed broad labels like "European," since they often contain more specific regional cuisines. On the other hand, I chose to keep "American" as a single category, combining its regional variations due to their overall similarity. The bar chart below shows the number of recipies per cuisine:


<iframe
 src="assets/recipes-per-cuisine.html"
 width="1000"
 height="400"
 frameborder="0"
 ></iframe>

The bar chart shows American cuisine is by far the most common, with over 9,000 recipes which is more than three times the next most frequent cuisines. Italian and Mexican cuisines follow, with around 2,500 and 2,100 recipes respectively. Beyond these top three, the remaining cuisines appear at much lower but relatively similar frequencies, suggesting that while the dataset is diverse, it has a strong bias toward American-style recipes.


### Bivariate Analysis

In addition to analyzing one variable at a time, I looked at relationships between two columns. Specifically, I grouped the data by cuisine and calculated the average for all of the nutrition metrics: calories, total fat, sugar, sodium, protein, saturated fat, and carbohydrates. These were then visualized using bar charts to help identify how different cuisines compare in terms of their typical nutritional profiles. This analysis was done to provide a more nuanced view of each cuisine's general nutritional characteristics and determine the health considerations of each cuisine. I decided to display the bar charts comparing the average values of calories, sodium, sugar, and protein.

**Calories**: This chart shows that Thai and Italian cuisines have the highest average calorie counts among all cuisines analyzed, while Canadian, Australian, and Indian cuisines tend to be the lowest. Interestingly, some commonly perceived "heavier" cuisines like American and Mexican fall closer to the middle of the range, rather than the top. This suggests that for those aiming to reduce calorie intake, it may be wise to limit dishes from Thai and Italian cuisines, or focus on lighter options within those categories.

<iframe
 src="assets/calories-per-cuisine.html"
 width="1000"
 height="400"
 frameborder="0"
 ></iframe>


**Sugar**: This chart shows that Caribbean and Thai cuisines have the highest average sugar content, both having an average of around 70g of sugar, while every other cuisine has a sugar content of less than 62g. For those looking to reduce sugar intake, it might be helpful to limit dishes from these two sweeter cuisines. Italian, Mexican, and Greek cuisines appear to have the lowest sugar levels of around 30g.

<iframe
 src="assets/sugar-per-cuisine.html"
 width="1000"
 height="400"
 frameborder="0"
 ></iframe>


**Sodium**: The chart shows that Thai, Japanese, and Chinese cuisines have the highest average sodium content. It is interesting to note that these are all Asian cuisines, suggesting a trend where Asian cuisines may generally have higher sodium levels compared to others. For those looking to reduce sodium intake, it might be helpful to limit dishes from these high-sodium cuisines or opt for recipes with less added salt. In contrast, Canadian, Indian, and Australian cuisines have the lowest sodium levels.

<iframe
 src="assets/sodium-per-cuisine.html"
 width="1000"
 height="400"
 frameborder="0"
 ></iframe>


**Saturated Fat**: French, Thai, Scandinavian, and Italian cuisines have the highest saturated fat levels. Since a high intake of saturated fats can lead to various health levels, it is advised to avoid cuisines with high saturated fat content. Chinese, Middle Eastern, and Japanese cuisines have the lowest saturated fat levels. 

<iframe
 src="assets/saturated-fat-per-cuisine.html"
 width="1000"
 height="400"
 frameborder="0"
 ></iframe>


After looking at the calories, sugar, sodium, and saturated fat in each cuisine, I found that Middle Eastern cuisine generally has the lowest amounts of all these factors, making it one of the healthier options overall.


### Interesting Aggregates

To better understand how recipe complexity might relate to nutritional content, I created ingredient bins based on the number of ingredients in each recipe. 

| ingredient_bin   |   calories |
|:-----------------|-----------:|
| 0-5              |    283.467 |
| 6-10             |    352.801 |
| 11-15            |    425.785 |
| 16-20            |    523.43  |
| 21-25            |    609.056 |
| 26-30            |    643.07  |
| 31+              |    710.764 |

From this table, it's clear that as the number of ingredients in a recipe increases, so does the average calorie content. This suggests that more complex recipes, which require more ingredients, are generally higher in calories compared to simpler ones. This could be due to the inclusion of higher-calorie ingredients, such as oils, sauces, or fats, as complexity increases. It helps highlight the potential for more intricate recipes with more ingredients to be less health-conscious in terms of calorie density, making it useful for consumers who are mindful of their caloric intake.

### Imputation

I didn’t perform imputation because the only column with missing data was avg_rating, which is missing when a recipe hasn’t been rated by any users. Since it makes sense for these values to be empty and there's no meaningful way to infer a rating that doesn’t exist, I chose not to apply any imputation.


## Framing a Prediction Problem
---

The question I pose is: **Given a recipe's macronutrient profile (total fat, sugar, sodium, protein, saturated fat, carbohydrates), number of ingredients, and the recipe's cuisine type, what would be the predicted number of calories?**

This is a **regression** problem because the target variable (calories) is continuous and numeric. At prediction time, which is before cooking begins, I will use:

**Numeric features:**
- Preparation data: `n_steps`, `n_ingredients`
- Nutrition data: `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, `carbohydrates`

**Categorical feature:**
- Tag data: `cuisine`

The goal is to learn how these inputs jointly predict the continuous outcome, `calories`.

I will evaluate model performance using **Mean Squared Error (MSE)** This metric helps quantify the difference between the predicted calorie values and the actual calorie values across all recipes. Since the goal of the project is to predict the number of calories in a recipe based on a variety of features like nutritional information and recipe details, MSE gives us a clear indication of how well the model is performing in terms of accuracy.


## Baseline Model
---

To build my baseline model, I used a linear regression pipeline that processes both numerical and categorical data. I first separated out the target variable, calories, and split the dataset into training and testing sets. I reserved 20% of my data for testing and used the remaining 80% for training. All numeric features—like total fat, sugar, sodium, protein, saturated fat, carbohydrates, number of ingredients, and number of steps—were standardized so that the model could treat them equally regardless of scale. For the categorical feature, cuisine tags were one-hot encoded to ensure they could be used effectively in the regression model.

These are the results I got after evaluating:

`Baseline MSE: 816.06`

`Baseline R²: 0.9908`


The baseline linear regression model achieved a Mean Squared Error (MSE) of 816.06 and an R² score of 0.9908 on the test set. The low MSE indicates that, on average, the model's predictions are very close to the actual calorie values, with relatively small errors when squared. This suggests that the model is effective at minimizing large deviations from true values, which is especially important when predicting nutritional metrics like calories.

An R² score of 0.9908 means that the model is able to explain over 99% of the variance in the calorie data. This high value suggests a strong linear relationship between the input features—nutritional components, ingredient and step counts, and the cuisine type—and the target variable. While promising, such a high R² might also hint at potential overfitting or the presence of highly predictive features (like macronutrients) that dominate the prediction. These results set a strong baseline and will be helpful for evaluating the value of more complex models later in the process.

Given these metrics, I would consider the baseline model to be “good” in terms of predictive accuracy and explanatory power. However, the extremely high R² also raises questions about whether the model may be overfitting to the training data or benefiting disproportionately from a few dominant features (e.g., total fat or carbohydrates). To assess this further, I plan to evaluate the model’s generalization ability on other datasets and compare it with more complex models. 

## Final Model
---

To improve upon the baseline linear regression model, I engineered two new features that incorporate more nuanced nutritional information and reflect real-world eating patterns:

`sugar_per_step`: This feature divides the sugar content by the number of recipe steps. The rationale behind this feature is that it captures how concentrated sugar is per unit of effort or complexity. Simpler recipes with high sugar content (like desserts or sugary drinks) often lead to high calorie counts, and this ratio helps the model account for that dynamic.

`carb_to_protein`: This feature calculates the ratio of carbohydrates to protein in a dish. I added this to capture the macronutrient balance, which plays a major role in determining calorie content. Meals that are heavy in carbs but low in protein—such as baked goods or starchy dishes—often have higher caloric density. By giving the model this ratio, it can more easily distinguish between lean, protein-dominant meals and calorie-dense carbohydrate-heavy meals.

I used these features in addition to the original numerical and categorical variables (standardizing continuous features and one-hot encoding cuisine). I also switched to a **Random Forest Regressor** for my final model to better capture non-linear relationships that a simple linear regression might miss. Random forests are well-suited for handling mixed data types and capturing interactions between features like macronutrient composition and cuisine type.

To optimize the RandomForestRegressor, I performed hyperparameter tuning using GridSearchCV. I focused on three key parameters:
- `n_estimators`: The number of trees (set to 50)
- `max_depth`: The maximum depth of the trees (tested with 10 and 20)
- `min_samples_split`: Minimum samples required to split a node (tested with 2 and 5)

The best parameters found were {`n_estimators`: 50, `max_depth`: 20, `min_samples_split`: 2}

With these settings, the final model achieved:

`Final MSE: 900.06`

`Final R²: 0.9899`

Although the final model's MSE is higher (indicating worse performance) and its R² slightly decreased compared to the baseline, it still explains a very high proportion of the variance in the target variable (calories). The model's R² of 0.9899 still indicates strong predictive power. In conclusion, the final model did not significantly outperform the baseline but benefited from feature engineering and hyperparameter tuning. The new features (sugar_per_step and carb_to_protein) provided relevant information, though they did not drastically improve the model.
