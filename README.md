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

To set up the data for it to be analyzed correctly, I made an is_restaurant variable that helps identify food and non-food related businesses, based on the business's category. I also made sure to standardize the missing values of the dataset; specifically the price column as many businesses had None in that column. I replaced the None with with NaN, to represent actual missing values, and this cleaning of the data made it easier to compare the food and non-food related businesses.

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

The histogram distribution of average business ratings is mainly between 4 and 5 stars. This shows that most businesses in Hawaii typically receive good reviews and positive feedback from customers, while almost no businesses receive ratings that are below 3 stars.

<iframe src="assets/num_reviews_dist.html" width="800" height="500" frameborder="0"></iframe>

From the histogram, we see how the number of reviews are strongly right-skewed, meaning the majority of businesses have relatively less reviews. Some businesses have a lot of reviews(over 500 reviews), but were removed from the graph to help the visualization be more clear.

**Bivariate Analysis:**

<iframe src="assets/food_vs_nonfood.html" width="800" height="500" frameborder="0"></iframe>

From the box plot, non-food-related businesses have slightly higher average ratings than the food-related businesses. This observation is not what I originally expected because I initially thought that restaurants and food-related businesses tend to receive higher ratings.

<iframe src="assets/reviews_vs_rating.html" width="800" height="500" frameborder="0"></iframe>

From the scatterplot, we see the relationship between the number of reviews and average ratings of businesses. The businesses with less reviews have a much wider range of ratings(1-5 stars), while the businesses with a higher numbers of reviews mainly receive ratings between 4 and 5 stars(much higher). This indicates that businesses with a higher number of reviews tend to receive higher ratings.

**Interesting Aggregates:**

| is_restaurant | avg_rating | avg_reviews | count |
|---|---|---|---|
| False | 4.37 | 114.12 | 17070 |
| True | 4.25 | 262.23 | 4437 |

Non food-related businesses have a higher average rating (4.37 stars) compared to food-related businesses (4.25 stars). Even though the food-related businesses receive a higher average number of reviews(262 reviews vs 114 reviews), they still receive lower average ratings. This shows how the restaurants and food-related businesses are reviewed by customers more often, but don't always receive better ratings. This observation is what led me to the hypothesis test that was done Step 4 of the project.

---

## Assessment of Missingness

Several different columns in the data set have missing values but the price column is likely NMAR. Many of the different businesses may decide to not report their price level on Google Maps on purpose. A reason for this could be because businesses may not want customers to see the price level; too expensive which may scare customers away, or too cheap which may look like the restaurant has poor quality. This missingness is not random, most likely depending on the business's own decision, which we cannot see in this dataset. This means the price column is likely NMAR (Not Missing At Random).

Other data that I could collect to determine if the price column was MAR, is to check other data posted on google maps, such as a website, menu, images, etc. as these could all help explain the missingness.

**Missingness Dependency:**

<iframe src="assets/missingness.html" width="800" height="500" frameborder="0"></iframe>

First test(price_missing vs avg_rating): p-value= 0.000000, observed difference= 0.173396. We reject the null hypothesis. The missingness of price IS dependent on avg_rating. Businesses that have missing prices on Google Maps tend to have higher average ratings than the businesses that report their price.

Second test(price_missing vs name_missing): p-value= 0.601100, observed difference= 0.000229. We fail to reject the null hypothesis. The missingness of price IS NOT dependent on if the name variable is missing. The business name has nothing to do with the price missing, so it makes sense since these are unrelated features.
---

## Hypothesis Testing

**Null Hypothesis**: Food-related businesses in Hawaii have the same average rating as non-food related businesses.

**Alternative Hypothesis**: Food-related businesses in Hawaii have higher average ratings than non-food related businesses.

**Test Statistic**: Difference in mean average ratings between food and non food-related businesses in Hawaii (food - non-food)

**Significance Level**: 5%

<iframe src="assets/hypothesis.html" width="800" height="500" frameborder="0"></iframe>

I performed a permutation test with 10,000 simulations in order to generate an empirical distribution of the test statistic under the null hypothesis. The p-value I got was 1.0000, with an observed difference in mean ratings of -0.1226. This means that the food-related businesses, received lower average ratings than non-food related businesses which is the opposite of my alternative hypothesis. Since the p-value(1.0000) is much larger then my 0.05 significance level, we fail to reject the null hypothesis. There is not enough evidence to conclude that food-related businesses in Hawaii, receive higher average ratings than the non-food-related businesses in Hawaii.

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