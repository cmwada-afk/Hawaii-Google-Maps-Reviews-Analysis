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

For the prediction problem my model will try to predict a business's average Google Maps rating(avg_rating), based on the different features that are available about the business such as location, number of reviews, etc. This will be a regression type problem, because avg_rating is the response variable, and it is a continuous variable that is always between 1 and 5 stars for the rating. avg_rating is the response variable that I chose because it directly measures the customers satisfaction and success because people leave the reviews when they feel strong about their opinion. The main focus of this project is the success of each business and predicting the ratings of the businesses can help us figure out the business features that are associated with higher customer satisfaction.

The evaluation metric I would use is the Root Mean Squared Error(RMSE), and I chose this over Mean Absolute Error(MAE), because the RMSE is more sensitive to outliers, meaning it will heavily penalize the bigger prediction errors calculated. This matters way more when we are trying to predict the average ratings of businesses because since the ratings are between 1-5 stars, a prediction that is off by 2 stars is super wrong, and needs to be penalized more then being off by 0.3 stars. The RMSE also will measure the prediction error in the same units as the response variable(1-5 stars) which is effective when solving a regression problem. The features are the number of reviews(num_of_reviews), category(is_restaurant), the price level(price), and the location geographically(latitude & longitude). I chose these features because when solving the predictions, it only makes sense to use features that would be available to use when predicting a business's ratings, and not features that are made after a customer visits a business.

---

## Baseline Model

The baseline model uses linear regression, using the features: num_of_reviews(quantitative, passthrough), is_restaurant(nominal, one-hot encoded), and price (nominal, one-hot encoded), where the steps were implemented in a sklearn pipeline. From the test on the model, the RMSE that was calculated is 0.4447. This means that the model was not that effective at predicting the businesses' ratings because the RMSE means it was wrong on average by about 0.45 stars. Being this off on a scale between 1 and 5 stars is not that effective, but since only 3 features were used on this linear regression model, there were limits to the predictions. There is a lot of improvements that are needed to be made in step 7, possibly adding more features or a different model.

---

## Final Model

For the final model, I added two more new engineered features: log_reviews (quantitative): logarithm of num_of_reviews and lat_lon_interaction(quantitative): the product of latitude and longitude.

I added the log_reviews feature because we saw from earlier in step 2, that the distribution of num_of_reviews is heavily right skewed; meaning most of the businesses have very small number of reviews while some businesses have over 500. This log transform helps fix the skew, where taking the log, reduces the influence of higher number of review businesses to help better represent the relationship between the number of reviews and ratings without getting dominated by outliers(focusing on less reviews).

I added the lat_lon_interaction feature because there are certain geographic location within Hawaii, like tourist cities like Waikiki, may be associated with higher or lower ratings because of the amount of people visiting. Different locations within Hawaii, can bring in different customers to a variety of different types of businesses. By multiplying both the latitude and longitude togehter, it creates a single feature that helps show the location based effect on the ratings.

For the final model I used a Random Forest Regressor, which improves from the baseline Linear Regression model by capturing some of the non-linear relationships between features and ratings.

The features I used are num_of_reviews (quantitative), log_reviews (quantitative, engineered), lat_lon_interaction (quantitative, engineered), latitude (quantitative), longitude (quantitative), is_restaurant (nominal, one-hot encoded), and price (nominal, one-hot encoded).

I used StandardScaler on quantitative features, and OneHotEncoder on the categorical features. All of the preprocessing and model training steps were implemented within a single sklearn Pipeline.

To improve the final model performance, I used GridSearchCV to help tune the Random Forest hyperparameters. The hyperparameters tested were:

n_estimators: [50, 100] — controls number of trees in the forest
max_depth: [5, 10, None] — controls how deep each tree can grow

Final Model Results:

The best hyperparameters that I found were n_estimators=50 and max_depth=5.

Baseline RMSE: 0.4447
Final Model RMSE: 0.4189
Improvement: 0.0258
The final model achieved a RMSE of 0.4189, while the original baseline model had a RMSE of 0.4447. This means that the final model improved by about 0.0258 stars(rating).

We can see that the final model ended up performing better than the baseline model because it added additional information(log_reviews and lat_lon_interaction) about possible review trends and the businesses' locations. The final model also used a more flexible machine learning algorithm that is capable of capturing the nonlinear relationships between business features and the customer ratings, which was not able to be done in the baseline. The Random Forest Regressor ended up performing better than the Linear Regression model because it was able to capture and show the non-linear relationships between features and ratings. The engineered features log_reviews and lat_lon_interaction helped the final model better understand the review volume and frequency patterns; and the geographic location effects on business ratings.

---

## Fairness Analysis

For the fairness analysis I will analyze whether the final model that I performed in step 7, performs equally well for food-related businesses compared to non-food related businesses, because this was the main question that influenced my project.

Group X: Food-related businesses (is_restaurant = True)
Group Y: Non-food related businesses (is_restaurant = False)
Evaluation Metric: RMSE
Null Hypothesis: The model is fair. The RMSE for food-related and non-food related businesses are roughly the same, and any differences are due to random chance.
Alternative Hypothesis: The model is unfair. The RMSE for food-related businesses is significantly different from the RMSE for non-food related businesses.
Significance Level: 0.05
Test Statistic: Difference in RMSE (food related RMSE - non-food related RMSE)

<iframe src="assets/fairness.html" width="800" height="500" frameborder="0"></iframe>

Fairness Results:

RMSE for food related businesses: 0.4162
RMSE for non-food related businesses: 0.4232
Observed difference: -0.0070
P-value: 0.8726

The p-value (0.8726) is much greater than the significance level of 0.05, so we fail to reject the null hypothesis. There is no significant evidence that the final model performs differently for food-related vs non-food related businesses. This means the model appears to be fair across both groups.