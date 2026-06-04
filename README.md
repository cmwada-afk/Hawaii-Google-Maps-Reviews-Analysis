# Successful Businesses in Hawaii Analysis

**By Chase Wada**

---

## Introduction

In this project, I analyzed Google Maps reviews of businesses in Hawaii to help answer the question: **What factors are associated with successful businesses in Hawaii?**

Hawaii is one of the most popular tourist destinations in the United States, meaning there are lots of different businesses, varying from stores, hotels, restaurants, etc. For the analysis of this dataset, business success is defined as their average rating received on Google Maps; represented by `avg_rating` in the dataset, which shows customer satisfaction. 
 
The focus question of this analysis is **Do food-related businesses receive higher average ratings than non-food-related businesses in Hawaii?** This question is important because people looking to open a business in Hawaii or other tourist destinations may want to understand why some types of businesses earn better customer ratings.

This dataset contains **21,507 businesses** and **1,504,347 reviews**.

Relevant columns:

| Column | Description |
|---|---|
| `avg_rating` | Average rating of a business(1-5 stars) |
| `num_of_reviews` | Total number of reviews |
| `category` | Business category(ex: Recreation center, Restaurant) |
| `price` | Price level  |
| `latitude` / `longitude` | Geographic location of the business |

---

## Data Cleaning and Exploratory Data Analysis

To clean the data, I created an `is_restaurant` column to identify food-related businesses based on their category. I also replaced `'None'` strings in the `price` column with actual `NaN` values to properly represent missing data.

**Cleaned DataFrame (first 5 rows):**

| name | category | avg_rating | num_of_reviews | price | is_restaurant |
|---|---|---|---|---|---|
| Hale Pops | [Restaurant] | 4.4 | 18 | None | True |
| SMP - Single Marine Program | [Recreation center] | 4.1 | 18 | None | False |
| 2 Cheesy Guys | [Food court] | 5.0 | 6 | None | True |
| Kraken Coffee Kahului | [Coffee shop] | 4.8 | 8 | $ | False |
| Akasatana Ramen Kyoto | [Ramen restaurant] | 5.0 | 1 | None | True |

**Univariate Analysis:**

<iframe src="assets/avg_rating_dist.html" width="800" height="500" frameborder="0"></iframe>

Most businesses in Hawaii receive ratings between 4 and 5 stars, showing that customers generally have positive experiences. Very few businesses receive ratings below 3 stars.

<iframe src="assets/num_reviews_dist.html" width="800" height="500" frameborder="0"></iframe>

The distribution of review counts is heavily right-skewed. Most businesses have relatively few reviews, while a small number have extremely large review counts.

**Bivariate Analysis:**

<iframe src="assets/food_vs_nonfood.html" width="800" height="500" frameborder="0"></iframe>

Non-food businesses have slightly higher average ratings than food-related businesses, which was surprising given my initial expectation.

<iframe src="assets/reviews_vs_rating.html" width="800" height="500" frameborder="0"></iframe>

Businesses with more reviews tend to have higher and more stable ratings, suggesting that established businesses perform better.

**Interesting Aggregates:**

| is_restaurant | avg_rating | avg_reviews | count |
|---|---|---|---|
| False | 4.37 | 114.12 | 17070 |
| True | 4.25 | 262.23 | 4437 |

Food-related businesses receive more reviews on average (262 vs 114) but have lower average ratings (4.25 vs 4.37).

---

## Assessment of Missingness

The `price` column is likely **NMAR** (Not Missing At Random). Businesses may deliberately choose not to report their price level on Google Maps — perhaps because they don't want to signal that they are too expensive or too cheap. This missingness depends on the business's own decision, which we cannot observe in the data. Additional data such as business websites or menus could help explain this missingness, making it MAR.

**Missingness Dependency:**

<iframe src="assets/missingness.html" width="800" height="500" frameborder="0"></iframe>

- **Test 1** (price ~ avg_rating): p-value = 0.0000, observed difference = 0.1734. The missingness of `price` IS dependent on `avg_rating`.
- **Test 2** (price ~ name_missing): p-value = 0.6008, observed difference = 0.000229. The missingness of `price` does NOT depend on whether `name` is missing.

---

## Hypothesis Testing

**Null Hypothesis**: Food-related businesses in Hawaii have the same average rating as non-food related businesses.

**Alternative Hypothesis**: Food-related businesses in Hawaii have higher average ratings than non-food related businesses.

**Test Statistic**: Difference in mean average ratings (food minus non-food)

**Significance Level**: 5%

<iframe src="assets/hypothesis.html" width="800" height="500" frameborder="0"></iframe>

The p-value was 1.0000, with an observed difference of -0.1226. Since the p-value is much greater than 0.05, we fail to reject the null hypothesis. There is not enough evidence to conclude that food-related businesses receive higher ratings — in fact, they receive lower ratings on average.

---

## Framing a Prediction Problem

**Prediction Problem**: Predict a business's `avg_rating` based on available features.

**Type**: Regression — `avg_rating` is continuous (1–5 stars).

**Response Variable**: `avg_rating` — directly measures customer satisfaction and business success.

**Evaluation Metric**: RMSE — chosen over MAE because it penalizes larger errors more heavily, which matters when predicting ratings on a 1–5 scale.

**Features used**: `num_of_reviews`, `is_restaurant`, `price`, `latitude`, `longitude` — all known before a customer visits the business.

---

## Baseline Model

The baseline model uses **Linear Regression** with three features:
- `num_of_reviews` (quantitative, passthrough)
- `is_restaurant` (nominal, one-hot encoded)
- `price` (nominal, one-hot encoded)

All steps were implemented in a single sklearn Pipeline.

**Baseline RMSE: 0.4447**

This model is not particularly good — being off by 0.44 stars on a 1–5 scale is a moderate error. There is significant room for improvement.

---

## Final Model

The final model uses a **Random Forest Regressor** with two engineered features:

- `log_reviews`: Log transform of `num_of_reviews` — fixes the right skew in review counts so the model isn't dominated by outliers.
- `lat_lon_interaction`: Product of latitude and longitude — captures location-specific effects on ratings, such as tourist-heavy areas like Waikiki.

**Hyperparameter tuning** via GridSearchCV (3-fold CV):
- Best `n_estimators`: 50
- Best `max_depth`: 5

| Model | RMSE |
|---|---|
| Baseline (Linear Regression) | 0.4447 |
| Final (Random Forest) | 0.4189 |
| Improvement | 0.0258 |

The Random Forest improved over Linear Regression by capturing non-linear relationships between features and ratings.

---

## Fairness Analysis

**Group X**: Food-related businesses (`is_restaurant = True`)

**Group Y**: Non-food businesses (`is_restaurant = False`)

**Null Hypothesis**: The model is fair. RMSE for food and non-food businesses are roughly the same.

**Alternative Hypothesis**: The model is unfair. RMSE for food businesses is higher.

**Significance Level**: 0.05

<iframe src="assets/fairness.html" width="800" height="500" frameborder="0"></iframe>

- RMSE food businesses: 0.4162
- RMSE non-food businesses: 0.4232
- Observed difference: -0.0070
- **P-value: 0.8726**

Since p-value (0.8726) > 0.05, we fail to reject the null hypothesis. The model appears to be fair across both groups.