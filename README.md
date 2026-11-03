# AmyloidB_Aggregation_ML
**A Hybrid Molecular Representation and Non-Linear Stacking Ensemble Framework for Predicting Amyloid $\beta$ ($\text{A}\beta_{42}$) Aggregation Inhibitors**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![RDKit](https://img.shields.io/badge/Cheminformatics-RDKit-green.svg)](https://www.rdkit.org/)

## Overview

**AmyloidB_Aggregation_ML** is a quantitative structure–activity relationship (QSAR) and machine learning pipeline designed to predict the anti-aggregation potency (pIC₅₀ / IC₅₀) of small molecules against Aβ42 fibrillation (Thioflavin T fluorescence assay).

The workflow integrates a **hybrid molecular featurization** scheme with a **two-layer StackingRegressor ensemble** and an **Applicability Domain (AD)** reliability assessment, enabling rapid and trustworthy virtual screening of novel multi-target-directed ligands (MTDLs) for Alzheimer's disease.

---

## Methodological Pipeline

### 1. Data Standardization & Preprocessing
* **SMILES Canonicalization:** Salts and counterions are stripped by retaining the largest molecular fragment, followed by canonical SMILES standardization via `RDKit`.
* **Activity Capping & Transformation:** Experimental IC₅₀ values (in nM) are capped at 40,000 nM (40 µM) and converted into logarithmic scale:

 $$ \text{pIC}_{50} = 9.0 - \log_{10}(\text{IC}_{50}\text{ [nM]}) $$

### 2. Hybrid Molecular Featurization
Each standardized molecule is encoded into a comprehensive hybrid feature vector combining:
1. **2D Physicochemical & Topological Descriptors:** Extracted from `rdkit.Chem.Descriptors` (extreme values $> 10^6$ or non-finite entries filtered).
2. **MACCS Structural Keys:** $167\text{-bit}$ substructure keys capturing key functional groups.
3. **Extended-Connectivity Fingerprints (ECFP4):** $2048\text{-bit}$ Morgan fingerprints with radius $= 2$.

### 3. Dual-Stream Feature Selection (`ColumnTransformer`)
To prevent data leakage during cross-validation, preprocessing and feature selection are encapsulated inside `scikit-learn` pipelines:
* **Continuous Descriptors Stream:** Median imputation (`SimpleImputer`) $\rightarrow$ zero-variance filtering (`VarianceThreshold(0.0)`) $\rightarrow$ outlier-robust scaling (`RobustScaler`) $\rightarrow$ Mutual Information selection (`SelectKBest(mutual_info_regression, k=50)`).
* **Binary Fingerprints Stream (MACCS + ECFP4):** Near-zero variance filtering (`VarianceThreshold(0.01)`) $\rightarrow$ Mutual Information selection (`SelectKBest(mutual_info_regression, k=50)`).

### 4. Non-Linear Stacking Architecture
* **Level-0 Base Estimators:**
  * **XGBoost Regressor (`XGBRegressor`):** `n_estimators=500`, `max_depth=4`, `learning_rate=0.02`, `subsample=0.8`, `colsample_bytree=0.7`, $L_1/L_2$ regularized (`reg_alpha=0.2`, `reg_lambda=1.5`).
  * **Random Forest Regressor (`RandomForestRegressor`):** `n_estimators=400`, `max_depth=12`, `min_samples_leaf=2`, `max_features=0.5`.
  * **Extra Trees Regressor (`ExtraTreesRegressor`):** `n_estimators=400`, `max_depth=14`, `min_samples_leaf=2`, `max_features=0.5`.
  * **Support Vector Regressor (`SVR`):** `StandardScaler` followed by RBF-kernel SVR (`C=2.5`, `epsilon=0.05`, `gamma='scale'`).
* **Level-1 Meta-Learner:**
  * Non-negative **Ridge Regression** (`Ridge(alpha=5.0, positive=True)`) trained on 5-fold out-of-fold (OOF) predictions to prevent overfitting and derive optimal ensemble weights.

---
## Installation

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/ryanlinhvu/AmyloidB_Aggregation_ML.git](https://github.com/ryanlinhvu/AmyloidB_Aggregation_ML.git)
   cd AmyloidB_Aggregation_ML
