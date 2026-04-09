# Body Fat Prediction Model
### Regression Research for Health & Wellness Applications

---

## Background

Accurate body fat estimation is a core input for personalized calorie targets and weight
management plans. Clinical methods like DEXA scans and hydrostatic weighing are accurate
but expensive, time-consuming, and inaccessible for most users of consumer health apps.

This project builds and evaluates a regression model that estimates body fat percentage
from simple physical measurements — things a user can enter themselves during app
onboarding: age, weight, height, and basic body circumference measurements. The goal is
a model accurate enough to drive meaningful calorie and macro recommendations without
requiring any clinical equipment.

---

## Business Problem

A health and wellness app needs to personalize calorie targets for three user goals:
weight loss, weight maintenance, and weight gain. Calorie targets depend on body
composition — specifically, how much of a user's weight is fat versus lean mass.
Without an accurate body fat estimate, calorie recommendations default to generic
BMI-based formulas that are known to be imprecise, especially for users who are
muscular or carry weight differently.

**This model is designed to replace the BMI baseline with a measurement-driven
body fat estimate, fed directly from user onboarding inputs.**

---

## Dataset

- **Source:** `fat` dataset, `faraway` R package
- **Observations:** 252 adult males
- **Response variable:** Body fat percentage (`siri`) — measured via underwater weighing
- **Predictors:** Age, weight, height, BMI, and 10 body circumference measurements
  (neck, chest, abdomen, hip, thigh, knee, ankle, biceps, forearm, wrist)
- **Excluded variables:** `brozek` and `density` — both are direct derivations of the
  target variable and would not be available in a real app context

---

## Approach

Five regression models are trained on 70% of the data and evaluated on a held-out
30% test set. Models are compared on R², MSE, and train-to-test stability.

| Model | Description |
|---|---|
| Linear Regression | Full model baseline with all predictors |
| Stepwise Selection | Automatic variable selection (both directions) |
| Ridge Regression | L2 regularization — handles multicollinearity |
| Lasso Regression | L1 regularization — built-in variable selection |
| Elastic Net | Combines Ridge and Lasso penalties |

A Huber robust regression is also included to assess outlier influence on model
stability — important for a production model that will encounter unusual body types.

---

## Key Findings

- **Ridge regression** achieved the best generalization: test R² of 0.876, MSE of 9.67
- All models trained near 97% R², but Linear and Stepwise dropped significantly on
  unseen data — confirming overfitting without regularization
- Severe multicollinearity exists across body measurements (VIF up to 199 for weight)
- Ridge handles correlated predictors better than any other method tested
- Two outlier observations were flagged via Huber weighting — the production model
  should route these edge cases to a fallback rather than returning a raw prediction

---

## Production Recommendation

**Deploy Ridge regression** as the body fat estimation engine. Key reasons:

- Highest accuracy on unseen data (test R² 0.876)
- Stable across the full range of body types in the dataset
- Handles correlated body measurements without requiring manual feature engineering
- All inputs are self-reportable during user onboarding — no clinical visit required

Body fat percentage from this model feeds directly into downstream calorie calculation:

```
Lean Body Mass  = Weight × (1 - BodyFat%)
Calorie Target  = BMR(LBM) × Activity Multiplier ± Goal Adjustment
```

---

## File Structure

```
bodyfat-prediction-model/
├── analysis.Rmd       # Full analysis — data prep, modeling, evaluation
├── requirements.R     # Package installation script
└── README.md
```

---

## Setup

```r
source("requirements.R")
```

Open `analysis.Rmd` in RStudio and knit to HTML or PDF.

---

## Requirements

- R 4.0+
- RStudio (recommended)
- See `requirements.R` for full package list
