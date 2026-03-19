# U.S. Power Outages: Causes, Patterns, and Predictions

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

**Combining date and time columns:** The dataset stored outage start and restoration times as separate date and time columns. These were combined into two proper datetime columns, `OUTAGE.START` and `OUTAGE.RESTORATION`, and the original split columns were dropped.

**Replacing zeros with NaN:** Values of 0 in `OUTAGE.DURATION`, `CUSTOMERS.AFFECTED`, and `DEMAND.LOSS.MW` were replaced with NaN. A duration of 0 minutes indicates either an instantaneous event or a missing value rather than a true observation, and treating it as a real value would skew any analysis of outage severity.

**Fixing column types:** The `MONTH` column was cast to a nullable integer since it contained NaN values that prevented it from being stored as a standard integer.

**Recomputing percentage columns:** Several columns storing customer percentage breakdowns contained raw Excel formula strings rather than computed values. These were recalculated directly from the source columns.

Here are the first few rows of the cleaned DataFrame:
| YEAR | MONTH | U.S._STATE | CAUSE.CATEGORY | OUTAGE.DURATION | CUSTOMERS.AFFECTED |
|-----:|------:|:-----------|:---------------|----------------:|-------------------:|
| 2011 | 7 | Minnesota | severe weather | 3060 | 70000 |
| 2014 | 5 | Minnesota | intentional attack | 1 | nan |
| 2010 | 10 | Minnesota | severe weather | 3000 | 70000 |
| 2012 | 6 | Minnesota | severe weather | 2550 | 68200 |
| 2015 | 7 | Minnesota | severe weather | 1740 | 250000 |


---



## Assessment of Missingness



---

## Hypothesis Testing



---

## Framing a Prediction Problem



---

## Baseline Model



---

## Final Model



---

## Fairness Analysis