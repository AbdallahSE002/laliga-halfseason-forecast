# La Liga First-Half Forecast (2025/26)

Predicting how many points each La Liga team will earn in the first half (matchweeks 1–19) of the 2025/26 season using only their first-half performance in 2024/25 and four prior seasons.

---

## 🚀 Problem Statement  
Can we learn a mapping  
> **first-half stats of season _N_** → **first-half points of season _N_ + 1**  
using only publicly available match data?  

Then, using 2024/25’s first-half features, forecast the first-half point totals for 2025/26.

---

## 📊 Data & Scope  
- **Source:** FBref “La Liga Matches 2019–2025” (Kaggle) → (Link: https://www.kaggle.com/datasets/marcelbiezunski/laliga-matches-dataset-2019-2025-fbref/data)
- **Constraint:** 2024/25 season only has rounds 1–26; we use rounds 1–19 (first half) and ignore the missing second half  
- **Season code:** end-year of campaign (e.g. 2024/25 → `season = 2024`)  

---

## 🛠️ Tech Stack  
- **Python 3.9+**  
- Data: `pandas`, `numpy`  
- Modeling: `scikit-learn` (LinearRegression, RandomForest), `TensorFlow / Keras`  
- Validation: `TimeSeriesSplit`, `RandomizedSearchCV`, `Wilcoxon signed-rank`  
- Visualization: `matplotlib`  

---

## 🔍 Pipeline Overview

All steps are implemented in **`La_Liga_2025_26_Half_Season_Prediction_Analysis.ipynb`**:

1. **Data Preparation**  
   - Load the raw CSV and drop irrelevant columns  
   - Compute `season` (campaign end‐year) and per‐team “round” index  
   - Aggregate first‐half (rounds 1–19) stats:  
     - Points (3/1/0 for W/D/L)  
     - Goal Difference, xG, xGA  
     - Shot accuracy, possession percentage  

2. **Cross-Season Target Building**  
   - Merge season _N_ first-half features → season _N_+1 first-half points  
   - Assemble the training table for seasons 2019/20 → 2023/24  

3. **Train / Validation / Back-Test Split**  
   - **Train:** seasons where `season < 2022` (i.e. 2019→20, 2020→21, 2021→22)  
   - **Validation:** `season == 2022` (2022→23)  
   - **Back-Test:** `season == 2023` (2023→24)  

4. **Modeling**  
   - **Linear Regression** baseline  
   - **Random Forest** with `TimeSeriesSplit` & `RandomizedSearchCV` for hyperparameter tuning  
   - **TensorFlow MLP** (32→16 ReLU layers → linear output)  

5. **Evaluation & A/B Testing**  
   - Compute MAE/RMSE on the back-test split  
   - Wilcoxon signed-rank test comparing RF vs. TF model errors  

6. **Forecasting**  
   - Input: 2024/25 first-half features (`season == 2024`)  
   - Output: predicted 2025/26 first-half point totals for all 20 teams
  
---

## 📈 Results Summary

| Model               | Back-Test MAE | RMSE  |
|--------------------:|-------------:|------:|
| Linear Regression   | 5.29         | 6.92  |
| Random Forest       | 4.61         | 5.98  |
| TensorFlow MLP      | 5.44         | 7.13  |

- **A/B test** (RF vs TF on 2023/24 H1):  
  - **p = 0.006** → TF MLP’s lower error on validation was significant, but RF achieved best back-test MAE.

---

## 🔮 2025/26 First-Half Forecast

Using the **Random Forest** (lowest back-test MAE):

| Rank | Team             | Predicted Points (H1) |
|-----:|------------------|----------------------:|
| 1    | Atlético Madrid  | 40.2                   |
| 2    | Real Madrid      | 39.6                   |
| 3    | Barcelona        | 37.3                   |
| …    | …                | …                      |
| 20   | Leganés          | 16.3                   |
