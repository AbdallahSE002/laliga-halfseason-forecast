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

1. **Data Prep** (`src/01_data_prep.py`)  
   - Load raw CSV, drop unused columns  
   - Assign `season` (Aug–May end-year) and per-team quarterly “round” index  
   - Compute **first-half** aggregates (rounds 1–19):  
     - `first_half_points` (3/1/0 for W/D/L)  
     - `GD_first_half`, `xG_first_half`, `xGA_first_half`  
     - `shot_acc_first_half`, `poss_first_half`  

2. **Cross-Season Target** (`src/02_train_models.py`)  
   - Merge season _N_ first-half features → season _N_ + 1 first-half **points**  
   - Build a DataFrame of `(features_N, points_N+1)` for seasons 2019→20 through 2023→24  

3. **Train / Validation / Back-Test**  
   - **Train:** seasons 2019/20→2021/22 (`season` < 2022)  
   - **Val:** season 2022/23 (`season` == 2022)  
   - **Back-Test:** season 2023/24 (`season` == 2023)  

4. **Modeling**  
   - **Baseline:** Linear Regression  
   - **Tree:** Random Forest with `TimeSeriesSplit(n_splits=3)` + `RandomizedSearchCV`  
   - **Deep:** TensorFlow MLP (32→16 ReLU layers → linear output)  

5. **Evaluation & A/B Testing** (`src/03_evaluate.py`)  
   - Compute **MAE**/RMSE on back-test (2023→24 H1)  
   - **Wilcoxon signed-rank** test comparing RF vs TF MAE  

6. **Forecast** (`src/04_forecast.py`)  
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
