# Case Study Title

Predicting Formula 1 Tyre Degradation and Lap-Time Performance

---

## Problem Statement

In Formula 1 racing, tyre condition strongly affects how fast a car can drive. As tyres get older during a stint, the rubber wears down and lap times become slower. This pace drop is called tyre degradation.

Race engineers face the challenge of deciding the best lap to call a driver in for a pit stop. Pitting too early wastes tyre life, while pitting too late causes the driver to lose valuable seconds on worn tyres. Because tyre wear depends on the rubber compound and varies across different circuits, teams need reliable, data-driven predictions of lap times to make effective pit-stop strategy decisions.

---

## Objectives

1. Collect an original dataset of race lap timing and tyre usage across all 22 Grand Prix sessions of the 2023 Formula 1 season by web scraping publicly accessible sources.
2. Clean and preprocess the raw data, perform exploratory data analysis, and engineer features that capture non-linear tyre wear and recent pace history.
3. Train and compare multiple machine learning regression models to predict lap duration in seconds, evaluating them against a session median baseline.
4. Identify the main drivers of tyre-related pace loss and translate the model findings into practical strategy recommendations for race engineers.

---

## Data Collection Source and Method

The data was collected by web scraping two publicly accessible websites without bypassing any login or private systems:

1. **OpenF1 Data Portal (https://openf1.org):** Used as the primary data source to scrape structured JSON pages containing lap timing, tyre stint details, and driver metadata for all 22 race sessions of the 2023 season.
2. **Formula 1 Official Results Website (https://www.formula1.com/en/results):** Used as a secondary source to cross-verify race names, session keys, round numbers, and calendar order.

Custom Python scripts using the `requests` and `pandas` libraries were used to collect and combine three raw data streams:
- **Laps Stream:** 24,479 raw lap records containing lap duration, sector times, speed trap figures, and pit flags.
- **Stints Stream:** 1,350 raw stint records containing tyre compound types (Soft, Medium, Hard), stint numbers, and initial tyre age.
- **Drivers Stream:** 439 raw driver records containing driver names, car numbers, and team names.

The three streams were merged on session key and driver number to produce an initial dataset of 24,604 rows across 24 columns. After removing in-laps, out-laps, wet weather laps, and extreme safety-car outliers, the final cleaned dataset contained 22,305 reliable race laps.

---

## Analytics Methods Used

1. **Data Preprocessing and Cleaning:**
   - Filtered out pit-in and pit-out laps (923 laps) and wet/intermediate compound laps (704 laps).
   - Removed extreme lap-time outliers above the 99.5th percentile per race session (124 laps).
   - Imputed missing numeric sector times and speed trap values using column medians.

2. **Feature Engineering:**
   - Polynomial tyre-age terms (quadratic and cubic tyre age) to model accelerating tyre degradation.
   - 3-lap rolling average lap duration (`rolling_mean_3`) and rolling standard deviation to capture recent car pace.
   - Stint progress percentage (`stint_pct`) and compound-age interaction features.
   - One-hot and target encoding for circuit session baselines and team names.

3. **Predictive Modeling:**
   - Train-test split was performed chronologically by race session (first 18 races for training with 18,319 laps, final 4 races for testing with 3,986 laps) to prevent data leakage.
   - Evaluated seven regression models: Linear Regression, Ridge Regression, Decision Tree, Random Forest, Gradient Boosting, XGBoost, and LightGBM.
   - Benchmarked all models against a naive session median baseline using Mean Absolute Error (MAE), Root Mean Squared Error (RMSE), and R-squared (R2).
   - Applied 5-fold cross-validation on the training set to verify stability.

4. **Model Interpretation:**
   - Permutation feature importance to rank the strongest predictors.
   - Partial dependence plots to examine compound-specific degradation curves.

---

## Key Results

- **Best Model:** Random Forest achieved the best overall performance on the unseen test set with an MAE of 3.10 seconds, an RMSE of 4.14 seconds, and an R-squared of 0.858.
- **Cross-Validation:** In 5-fold cross-validation, Random Forest achieved an average MAE of 0.90 seconds (+/- 0.20 seconds), confirming that the model generalizes well across different race rounds.
- **Linear Models vs Tree Ensembles:** Linear Regression and Ridge Regression failed (R2 = -0.09) because tyre degradation is non-linear. Tree-based ensembles (Random Forest, Gradient Boosting, XGBoost) successfully captured compound-specific wear curves.
- **Top Predictors:** Recent rolling pace history (`rolling_mean_3`), individual sector times (S1, S2, S3), and tyre age were the most important features for predicting current lap duration.
- **Degradation Acceleration Points:** Soft and Medium tyre compounds showed rapid degradation acceleration beyond approximately 12 laps of tyre age, while Hard tyres showed consistent, flat degradation over long stints (up to 35-45 laps).

---

## References

1. OpenF1, "OpenF1 API - Free and open-source F1 data," [Online]. Available: https://openf1.org. [Accessed: Sep. 2026].
2. Formula One World Championship Limited, "Formula 1 Race Results," [Online]. Available: https://www.formula1.com/en/results. [Accessed: Sep. 2026].
3. F. Pedregosa et al., "Scikit-learn: Machine learning in Python," Journal of Machine Learning Research, vol. 12, pp. 2825-2830, 2011.
4. T. Chen and C. Guestrin, "XGBoost: A scalable tree boosting system," in Proc. 22nd ACM SIGKDD Int. Conf. Knowledge Discovery and Data Mining, 2016, pp. 785-794.
5. G. Ke et al., "LightGBM: A highly efficient gradient boosting decision tree," in Advances in Neural Information Processing Systems, vol. 30, 2017.
6. A. Heilmeier, M. Graf, J. Betz, and M. Lienkamp, "Application of Monte Carlo methods to consider probabilistic effects in a race simulation for circuit motorsport," Applied Sciences, vol. 10, no. 12, Art. no. 4229, 2020.
