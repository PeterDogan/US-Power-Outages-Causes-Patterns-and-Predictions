**By Peter Dogan**

---

## Introduction


Power outages are a constant reality for millions of Americans, yet the factors that determine how long they last and what causes them are not always well understood. This project analyzes a dataset of major power outage events across the continental United States from January 2000 to July 2016.

**Has the leading cause of major power outages shifted over the decades?**

Understanding how outage causes have changed over time matters for utility companies, policymakers, and everyday people who depend on reliable electricity. The importance of understanding power outage patterns and frequency is becoming an increasingly important policy issue. If severe weather continues to dominate, that points to a need for climate resilience infrastructure.

The dataset contains **1,540 rows**, where each row represents a single major power outage event. The relevant columns are:

| Column | Description |
|---|---|
| `YEAR` | The year the outage happened |
| `CAUSE.CATEGORY` | The broad category of cause  |
| `OUTAGE.DURATION` | How long the outage lasted in minutes |
| `CLIMATE.REGION` | The U.S. climate region where the outage happened |
| `U.S._STATE` | The state where the outage happened |
| `MONTH` | The month the outage started |
| `ANOMALY.LEVEL` | The climate anomaly level at the time of the outage |

---

## Data Cleaning and Exploratory Data Analysis

The raw Excel file contained 5 rows of metadata before the actual column headers, as well as a units row immediately after the header. These were skipped on load using the `header` and `skiprows` parameters.

The following cleaning steps were performed:

**Combining date and time columns:** The dataset stored outage start and restoration times as separate date and time columns. These were combined into two proper datetime columns, `OUTAGE.START` and `OUTAGE.RESTORATION`.

**Replacing zeros with NaN:** Values of 0 in `OUTAGE.DURATION`, `CUSTOMERS.AFFECTED`, and `DEMAND.LOSS.MW` were replaced with NaN.

**Fixing column types:** The `MONTH` column was cast to a nullable integer since it contained NaN values.

**Recomputing percentage columns:** Several columns storing customer percentage breakdowns contained raw Excel formula strings rather than computed values. 

Here are the first few rows of the cleaned DataFrame:

| YEAR | MONTH | U.S._STATE | CAUSE.CATEGORY | OUTAGE.DURATION | CUSTOMERS.AFFECTED |
|-----:|------:|:-----------|:---------------|----------------:|-------------------:|
| 2011 | 7 | Minnesota | severe weather | 3060 | 70000 |
| 2014 | 5 | Minnesota | intentional attack | 1 | nan |
| 2010 | 10 | Minnesota | severe weather | 3000 | 70000 |
| 2012 | 6 | Minnesota | severe weather | 2550 | 68200 |
| 2015 | 7 | Minnesota | severe weather | 1740 | 250000 |

### Univariate Analysis

<iframe
  src="assets/cause-count.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

Severe weather is the most common cause of major outages, accounting for nearly half of all events in the dataset. Intentional attacks are the second most frequent cause, with all other categories occurring far less often.

### Bivariate Analysis

<iframe
  src="assets/cause-proportion.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

While severe weather consistently dominates as the leading cause of outages across all years, intentional attacks grow substantially as a proportion of outages from 2011 onward, suggesting a meaningful shift in the landscape of outage causes over time.

### Interesting Aggregates

<iframe
  src="assets/heatmap.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

This heatmap shows the count of outages by cause category for each year. Severe weather is consistently the darkest column across all years, but intentional attacks become notably more frequent from 2011 onward.

---



## Assessment of Missingness

### MNAR

The `CUSTOMERS.AFFECTED` column is likely **MNAR**. The number of customers affected is most likely to go unreported when the outage was minor or localized, meaning the very outages with low customer impact are the ones least likely to have that impact recorded. The missingness therefore depends on the value of the missing data itself. To make this column MAR instead, additional data about the reporting practices of each utility company would be needed.

### Missingness

To analyze the missingness of `CUSTOMERS.AFFECTED`, two permutation tests are performed.

The first test examined whether missingness depends on `CAUSE.CATEGORY`. Using TVD as the test statistic, the observed TVD between the cause distributions when `CUSTOMERS.AFFECTED` is missing versus not missing was compared against 1,000 permutations of shuffled missingness labels.

The second test examined whether missingness depends on `OUTAGE.DURATION`. Using absolute difference in means as the test statistic, the mean duration when `CUSTOMERS.AFFECTED` is missing was compared to when it is not missing.

<iframe
  src="assets/missingness.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

The missingness of `CUSTOMERS.AFFECTED` depends on `CAUSE.CATEGORY` which yielded a p-value of 0.000, and the test against `OUTAGE.DURATION` yielded a p-value of 0.1710. 

---

## Hypothesis Testing

**Null Hypothesis:** The mean outage duration is the same before and after 2010. Any observed difference is due to random chance.

**Alternative Hypothesis:** The mean outage duration after 2010 is different from before 2010.

**Test Statistic:** Difference in group means. 

**Significance Level:** 0.05

**Result:** The observed difference in mean outage duration before and after 2010 was computed and compared against 10,000 permutations of shuffled group labels. The resulting p-value was 0.0000, meaning none of the 10,000 simulated differences were as extreme as the one observed in the real data.

**Conclusion:** At the 0.05 significance level, we reject the null hypothesis. The difference in mean outage duration before and after 2010 is unlikely to be due to random chance alone. 

---

## Framing a Prediction Problem


The prediction problem is to predict **`OUTAGE.DURATION`**. This is a **regression** problem since outage duration is a 
continuous numerical variable.

**Why this response variable?** Outage duration is one of the most 
important aspects of a power outage. It directly determines how disruptive an event 
is for affected customers, and understanding what factors drive longer outages can 
help utility companies prioritize response efforts.

**Features available at time of prediction:** Only features that would be known at 
the moment an outage begins are used. These include `CAUSE.CATEGORY`, 
`CLIMATE.REGION`, `MONTH`, `YEAR`, and `ANOMALY.LEVEL`. 

**Evaluation Metrics:** RMSE is used to evaluate the model. 
RMSE is appropriate here because it penalizes large prediction errors more heavily 
than small ones. R² is also 
reported as a secondary metric to measure how much variance the model explains.

---

## Baseline Model


The baseline model is a Linear Regression trained on four features using a single 
sklearn Pipeline.

The features used are `CAUSE.CATEGORY` and `CLIMATE.REGION`, both nominal categorical 
variables that were one-hot encoded since they have no natural ordering. `MONTH` and 
`YEAR` were passed through as numerical values since they are already in a usable format.

On the training set it achieved an RMSE of  5,190 minutes and an R² of 0.18, 
and on the test set an RMSE of 6,367 minutes and 
an R² of 0.17. This means predictions are off by over 4 days on average, and the 
model only explains about 17% of the variance in outage duration.

Outage duration is 
highly variable and a simple linear model with only cause and region information 
cannot capture the full complexity of what drives how long an outage lasts. The 
relatively small gap between training and test performance suggests the model is 
not overfitting so much as it is simply not capturing enough signal from the 
features it has access to.

---

## Final Model


Three new features were added on top of the baseline model.

`IS_SUMMER` is a binary flag for outages occurring in June, July, or August. The EDA 
showed a clear spike in outages during summer months driven by thunderstorms and heat 
waves. Summer outages tend to be more widespread and harder to resolve quickly, so 
flagging this season gives the model an explicit signal about conditions that drive 
longer durations.

`IS_WINTER` is a binary flag for January and February, which showed a secondary 
outage peak from ice storms and wind events. These types of outages also tend to last 
longer due to dangerous repair conditions, so separating them from the rest of the 
year adds meaningful signal.

`ANOMALY.LEVEL` captures how extreme the climate conditions were at the time of the 
outage. More extreme anomalies likely contribute to longer and harder to resolve 
outages, particularly for severe weather events.

A Random Forest Regressor was chosen over Linear Regression because outage duration 
has a non-linear relationship with its predictors. A linear model can't explain 
interactions between cause category and seasonal conditions exactly where a random forest 
excels.

Hyperparameters were tuned using GridSearchCV with 3-fold cross validation. The best 
parameters found were a max depth of 10, a minimum samples per leaf of 5, and 100 
estimators.

The final model improved over the baseline on both metrics. Test RMSE dropped from 
6,367 minutes to 6,038 minutes, and test R² improved from 0.17 to 0.26, meaning the 
final model explains about 26% of the variance in outage duration compared to 17%.

---

## Fairness Analysis


**Group X:** Severe weather outages

**Group Y:** Non-severe weather outages

**Evaluation Metric:** RMSE

**Null Hypothesis:** The model is fair. Its RMSE for severe weather outages and 
non-severe weather outages are roughly the same, and any observed difference is 
due to random chance.

**Alternative Hypothesis:** The model is unfair. Its RMSE differs between severe 
weather and non-severe weather outages.

**Test Statistic:** Absolute difference in RMSE between the two groups.

**Significance Level:** 0.05

The model achieved an RMSE of 4,046 minutes on severe weather outages and 7,417 
minutes on non-severe weather outages, an observed difference of 3,371 minutes. 
A permutation test with 1,000 shuffles yielded a p-value of 0.299.

At the 0.05 significance level, we fail to reject the null hypothesis. While the 
model does predict severe weather outages more accurately, that gap is not 
statistically significant.