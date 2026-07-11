# Task 2 — ML Fundamentals

## Part 1: Linear Regression from Scratch

Generated a synthetic dataset (`y = 4 + 3X + noise`, 100 samples) and implemented Linear Regression using Gradient Descent from scratch in NumPy — no sklearn model classes.

Tested learning rates from 0.001 to 0.5. At lr=0.5 the loss diverged instead of converging, which showed up clearly on the loss-vs-epochs plot. lr=0.1 with 1000 epochs converged cleanly and recovered weights close to the true values (w≈3, b≈4). Train and test metrics (MAE, MSE, RMSE, R²) were close to each other, which is expected since the data is genuinely linear with no real overfitting risk.

**Bonus (Polynomial Regression, degree 2):** the learned coefficient for the squared term shrank close to zero, and performance barely changed from the plain linear fit. Makes sense — the underlying data has no real quadratic relationship, so the model correctly found there wasn't one to fit.

## Part 2: Regularization on Ames Housing

Dataset: [Ames Housing](https://www.kaggle.com/datasets/shashanknecrothapa/ames-housing-dataset), ~2900 rows, 82 original columns.

**Missing values** needed column-by-column judgment rather than a blanket fill. Columns like Pool QC, Alley, Fence, Misc Feature, and Fireplace Qu had NA meaning "feature doesn't exist," so those were filled with `"None"`. Garage and Basement columns were trickier: most NAs meant no garage/basement, but a handful of rows had a real garage or basement with just one field unrecorded (e.g. a garage with cars and area filled in but quality missing). Those got mode-imputed instead of "None," since treating them the same way would have wrongly erased real data. Lot Frontage was filled using each house's neighborhood median (falling back to a global median where a neighborhood had no recorded values at all), since a house can't realistically have zero street frontage.

`SalePrice` was right-skewed, so the target was log-transformed (`log1p`) before training — this keeps a few very expensive houses from dominating the squared-error loss.

Categorical variables were one-hot encoded, which pushed the feature count past 200. Numeric features were scaled with StandardScaler, fit on the training split only.

**Baseline (SGDRegressor):** used instead of plain LinearRegression, since LinearRegression solves the normal equation directly, which gets expensive and less stable with 200+ features, while SGDRegressor uses iterative gradient descent — the same idea as Part 1, just scaled up. In practice it turned out very sensitive to learning rate on this high-dimensional data; even after tuning `eta0` and the learning rate schedule, it never converged to a stable, well-fit solution. That's a legitimate result on its own — it shows why the regularized models below are the more practical choice here.

**Ridge, Lasso, ElasticNet** were tuned with cross-validation (`RidgeCV`, `LassoCV`, `ElasticNetCV`) to pick alpha automatically. Results on the log-transformed target:

| Model | Test MAE | Test RMSE | Test R² |
|---|---|---|---|
| Ridge | 0.0064 | 0.0096 | 0.9155 |
| Lasso | 0.0072 | 0.0104 | 0.9001 |
| ElasticNet | 0.0063 | 0.0101 | 0.9064 |

Ridge performed best. Lasso zeroed out a meaningful number of coefficients — with 200+ one-hot columns, plenty of them (rare neighborhood categories, for instance) carry little real signal, and Lasso's penalty is built to drop exactly those.

## Part 3: Logistic Regression on Santander

Dataset: [Santander Customer Transaction Prediction](https://www.kaggle.com/competitions/santander-customer-transaction-prediction), 200k rows, 200 anonymized features, no missing values.

The target is imbalanced (~90% class 0, ~10% class 1), confirmed with `value_counts(normalize=True)` before doing anything else. Split with `stratify=y` to keep that ratio consistent between train and test, and scaled features with StandardScaler since logistic regression's regularization is scale-sensitive.

Trained with `class_weight='balanced'` to stop the model from just defaulting to the majority class. Swept C across [0.01, 0.1, 1, 10, 100] — accuracy and ROC-AUC barely moved (ROC-AUC held flat at 0.8599 across the whole range). With 160k+ training rows against 200 features, there's little room to overfit in the first place, so regularization strength doesn't have much to push against here — unlike Part 2, where the rows-to-features ratio was far tighter.

Final metrics on the test set:

| Metric | Class 0 | Class 1 |
|---|---|---|
| Precision | 0.97 | 0.29 |
| Recall | 0.78 | 0.78 |
| F1 | 0.87 | 0.42 |

Overall accuracy: 0.78, ROC-AUC: 0.86.

Accuracy actually dropped compared to a naive "always predict 0" baseline (~90%), but the model is more useful this way — without balancing, it would ignore the minority class almost entirely. The tradeoff here (catching 78% of real transactions at the cost of more false positives) is usually the right one for this kind of prediction task, where missing an actual positive tends to cost more than a false alarm.
