# INFO 648 Final Project: West South Central Neighborhood Growth Prediction

## Project Overview

This repository contains the final project for **INFO 648 – Neighborhood Trajectory Prediction with U.S. Census Data**. The project follows the CRISP-DM process to predict whether census tracts gained population from 2010 to 2020, then applies the trained pipeline to forecast which tracts are likely to grow from 2020 to 2030.

The assigned Census division for this project is **West South Central**, which includes:

- Arkansas
- Louisiana
- Oklahoma
- Texas

The business scenario is that the U.S. Department of Economic Development wants to allocate federal funding toward areas where population growth is likely heading. The model is designed to support that decision by identifying the states and tracts with the strongest predicted growth patterns.

## Repository Contents

```text
.
├── README.md
├── West_South_Central_Final_Project.ipynb
├── West_South_Central_Final_Project_Deck.pptx
├── student_tracts_raw (1).csv
├── forecast_tracts_2020.csv
├── data_dictionary.csv
├── region_tract_counts.csv
├── west_south_central_forecast_summary.csv
└── west_south_central_tract_forecast.csv
```

### Main Files

| File | Purpose |
|---|---|
| `West_South_Central_Final_Project.ipynb` | Main Jupyter notebook organized by the six CRISP-DM phases. |
| `West_South_Central_Final_Project_Deck.pptx` | Slide deck for the recorded presentation. |
| `student_tracts_raw (1).csv` | Training data with 2010 tract characteristics and 2020 population used to create the historical target. |
| `forecast_tracts_2020.csv` | 2020 tract data used for the 2020-to-2030 forecast. |
| `data_dictionary.csv` | Column descriptions for the provided data. |
| `region_tract_counts.csv` | Census division tract counts used to confirm the assigned region. |
| `west_south_central_forecast_summary.csv` | State-level forecast summary created by the notebook. |
| `west_south_central_tract_forecast.csv` | Tract-level forecast output created by the notebook. |

## Data Source

The data was provided for the INFO 648 final project. It is drawn from **IPUMS NHGIS time-series Census data** and standardized so that 2010 and 2020 values use the same 2010 census-tract boundaries. No outside data was downloaded for this project.

The training file uses 2010 demographic, housing, density, and settlement-type information to predict whether each tract gained population from 2010 to 2020. The 2020 forecast file contains the same feature structure measured in 2020 and is used only after the model is trained.

## Software and Libraries

The notebook uses Python and the course-approved libraries and methods used throughout the semester:

- `pandas`
- `numpy`
- `matplotlib`
- `scikit-learn`
  - `train_test_split`
  - `MinMaxScaler`
  - `OneHotEncoder`
  - `ColumnTransformer`
  - `Pipeline`
  - `KMeans`
  - `DecisionTreeClassifier`
  - `RandomForestClassifier`
  - `confusion_matrix`
  - `classification_report`
  - `roc_auc_score`
  - `RocCurveDisplay`

## How to Run the Notebook

1. Clone or download this repository.

2. Confirm the following input files are in the same folder as the notebook:

   ```text
   student_tracts_raw (1).csv
   forecast_tracts_2020.csv
   data_dictionary.csv
   region_tract_counts.csv
   ```

   The notebook also supports `student_tracts_raw.csv` if the local file does not include the `(1)` suffix.

3. Open the notebook:

   ```text
   West_South_Central_Final_Project.ipynb
   ```

4. Run all cells from top to bottom.

5. The notebook will generate two output CSV files:

   ```text
   west_south_central_forecast_summary.csv
   west_south_central_tract_forecast.csv
   ```

## Reproducibility

A fixed random seed is used throughout the notebook:

```python
RANDOM_SEED = 42
```

The train/test split, k-means model, decision tree, and random forest are all built using this seed so the analysis can be reproduced when the notebook is run from top to bottom.

## CRISP-DM Project Structure

The notebook is organized into the six CRISP-DM phases required for the project.

### Phase 1: Business Understanding

The project frames the Department of Economic Development's funding question as a supervised-learning problem. The target is whether a census tract gained population from 2010 to 2020.

Target definition:

```text
grew = 1 if growth_pct > 0
grew = 0 if growth_pct <= 0
```

The primary success criterion is to beat both the majority-class baseline and chance-level AUC by at least 0.10.

### Phase 2: Data Understanding

The notebook filters the national dataset to the assigned West South Central region and reviews:

- tract counts by state
- missingness and undefined calculations
- target class balance
- average growth by settlement type
- average growth by state

### Phase 3: Data Preparation

The preparation phase includes:

- filtering to Arkansas, Louisiana, Oklahoma, and Texas
- dropping tracts with 2010 population below 100
- dropping zero-area tracts
- converting tract counts into per-tract rates
- encoding state and settlement type as categorical variables
- creating the binary growth target
- creating the train/test split before any model fitting

The minimum population cutoff is used because very small tracts can produce unstable growth percentages.

### Phase 4: Modeling

The modeling phase includes two connected steps:

1. **K-means clustering** is fit on the training data only to group tracts into neighborhood types.
2. The cluster label is added back to the supervised-learning dataset as an engineered categorical feature.

Two classifiers are trained:

- Decision Tree
- Random Forest

Each classifier is trained twice:

- with the k-means cluster feature
- without the k-means cluster feature

This creates the required ablation test to determine whether the cluster feature improves performance.

### Phase 5: Evaluation

The evaluation phase reports:

- confusion matrix
- accuracy
- precision
- recall
- ROC curve
- AUC
- feature importance
- cluster ablation comparison
- leakage discussion

AUC is treated as the primary metric because the agency needs to rank tracts and states by likelihood of growth for funding prioritization.

Current model performance from the completed run:

| Model | Accuracy | Precision | Recall | AUC |
|---|---:|---:|---:|---:|
| Decision Tree with Cluster | 0.709 | -- | 0.804 | 0.770 |
| Random Forest with Cluster | 0.763 | 0.774 | 0.853 | 0.829 |
| Decision Tree without Cluster | -- | -- | -- | 0.772 |
| Random Forest without Cluster | -- | -- | -- | 0.829 |

The Random Forest with the cluster feature was selected for the forecast because it had the strongest overall performance among the cluster-enabled models.

The ablation result showed that the cluster feature did not materially improve the Random Forest. This is reasonable because the Random Forest can already model nonlinear relationships and feature interactions from the original rate variables.

### Phase 6: Deployment and Forecast

The forecast phase applies the frozen 2010-trained pipeline to the 2020 forecast file. The same feature engineering process, scaler, k-means model, and trained classifier are used to predict which tracts are likely to grow from 2020 to 2030.

The forecast is summarized at the state level because the Department of Economic Development allocates funding by state.

Current forecast summary:

| State | Forecast Tracts | Share Predicted to Grow | Average Growth Probability |
|---|---:|---:|---:|
| Texas | 5,216 | 76.51% | 62.21% |
| Oklahoma | 1,046 | 57.65% | 50.84% |
| Louisiana | 1,125 | 49.69% | 47.75% |
| Arkansas | 684 | 48.25% | 47.53% |

## Main Recommendation

The Department of Economic Development should prioritize Texas first for growth-sensitive funding review, followed by Oklahoma. Texas has the highest forecasted share of tracts predicted to grow, while Oklahoma is also above the regional midpoint.

The model should be used as a screening and prioritization tool, not as an automatic funding decision rule. The model identifies growth patterns, but it does not prove that any individual feature causes population growth.

## Key Assumption and Limitation

The 2020-to-2030 forecast assumes that the relationship between baseline neighborhood structure and next-decade growth observed from 2010 to 2020 will continue during the 2020s.

That assumption may not fully hold because:

- 2020 Census values were affected by the COVID-19 period
- the 2020 population values in the project data are interpolated estimates
- housing markets, migration patterns, and regional economic conditions may change over time
- neighboring tracts are spatially related, so a random train/test split may produce optimistic performance

A stronger future validation approach would be to test the model using county-held-out or state-held-out validation to better measure spatial generalization.

## Leakage Controls

The notebook controls leakage in the following ways:

- the train/test split is created before fitting the scaler, k-means, or classifiers
- the scaler is fit on training data only
- k-means is fit on training data only
- test and forecast tracts receive cluster labels from the train-fitted k-means model
- 2020 demographic variables are not used as training inputs
- the 2020 file is used only for the final forecast step

## AI Assistance Disclosure

AI assistance was used as a coding and aid to help structure the notebook and README.

## Project Deliverables

This repository supports the required final project deliverables:

1. GitHub repository with notebook, README, and data explanation
2. Jupyter notebook that runs from top to bottom with a fixed random seed
3. Slide deck for the recorded presentation
4. Recorded presentation walking through the pipeline, ablation result, and 2030 forecast
