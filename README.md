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

### 1. Molecular Data Preprocessing
* **SMILES Canonicalization:** Salts and counterions are stripped by retaining the largest molecular fragment, followed by canonical SMILES standardization via `RDKit`.
* **Activity Capping & Transformation:** Experimental IC₅₀ values (in nM) are capped at 40,000 nM (40 µM) and converted into logarithmic scale:

 $$ \text{pIC}_{50} = 9.0 - \log_{10}(\text{IC}_{50}\text{ [nM]}) $$

### 2. Hybrid Molecular Representation
Each standardized molecule is encoded into a comprehensive hybrid feature vector combining:
1. **2D Physicochemical & Topological Descriptors:** Extracted from `rdkit.Chem.Descriptors` (extreme values $> 10^6$ or non-finite entries filtered).
2. **MACCS Fingerprints:** $167\text{-bit}$ structural keys.
3. **ECFP4 Fingerprints:** $2048\text{-bit}$ Morgan fingerprints with radius $= 2$.

### 3. Feature Selection and Preprocessing
A dual-stream feature selection strategy was implemented using scikit-learn pipelines. Continuous descriptors underwent imputation, variance filtering, robust scaling, and mutual information-based selection. Binary fingerprints were processed using variance filtering and mutual information-based selection.

All preprocessing and feature selection steps were integrated into the modeling pipelines to minimize data leakage during cross-validation.
### 4. Stacking Ensemble Regression
A two-level stacking architecture was developed to predict pIC₅₀ values.
* **Base learners (Level 0):**
  * XGBoost Regressor (`XGBRegressor`)
  * Random Forest Regressor (`RandomForestRegressor`)
  * Extra Trees Regressor (`ExtraTreesRegressor`)
  * Support Vector Regressor (`SVR`)
* **Meta-Learner (Level 1):**
  * A non-negative Ridge Regression model was trained using five-fold out-of-fold predictions to combine the outputs of the base learners and reduce overfitting.

Detailed model hyperparameters and implementation settings are provided in the corresponding configuration files.

---
## Installation

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/ryanlinhvu/AmyloidB_Aggregation_ML.git](https://github.com/ryanlinhvu/AmyloidB_Aggregation_ML.git)
   cd AmyloidB_Aggregation_ML
