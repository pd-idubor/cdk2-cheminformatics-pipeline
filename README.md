# CDK2 Cheminformatics Workflow

A predictive cheminformatics pipeline targeting Cyclin-dependent kinase 2 (CDK2). The workflow curates public bioactivity records from ChEMBL, evaluates QSAR regression models with scaffold-aware validation, defines a quantitative applicability domain (AD) to flag unreliable extrapolations, and prepares a model for virtual screening of natural products.

> **Status:** Work in progress (Notebooks 01–04 complete).

---

## Project Specifications

| Item | Detail |
|---|---|
| Target | Cyclin-dependent kinase 2 (*Homo sapiens*) |
| UniProt | [P24941](https://www.uniprot.org/uniprotkb/P24941) |
| ChEMBL Target | [CHEMBL301](https://www.ebi.ac.uk/chembl/target_report_card/CHEMBL301/) |
| Data Endpoint | IC50 (nM, standard relation "="), converted to pIC50 (M) |
| Screening Library | *To be added* |

## Pipeline Overview

- [x] **1. Data curation (`01`):** Programmatic retrieval of CDK2 IC50 records from ChEMBL, cleaning, and logged record of filtering steps.
- [x] **2. Chemical space and SAR analysis (`02`):** Drug-likeness, PAINS structural alerts, Murcko scaffolds, chemical space maps, and scaffold-based train/holdout split.
- [x] **3. QSAR modelling & applicability domain (`03–04`):** Feature and algorithm benchmarking (Morgan fingerprints vs. RDKit descriptors; Random Forest vs. XGBoost) under random and scaffold cross-validation, holdout evaluation, and (nearest-neighbor Tanimoto) applicability domain definition.
- [ ] **4. Virtual screening (`05`):** Prediction on a natural products library, flagging unreliable predictions (`similarity < 0.40`), and inspecting structural alerts among top hits.
- [ ] **5. Molecular docking (`06`):** Docking of shortlisted screening candidates against a CDK2 crystal structure.

## Results Summary

### 1. Data Curation (Notebook 01)

| Curation Step | Remaining Records / Molecules |
|---|---|
| ChEMBL IC50 records (exact values, binding assays, confidence score ≥ 8, duplicates removed) | 2,763 records |
| After removal of records with missing structures | 2,759 records (2,104 molecules) |
| Structure standardization and aggregation of repeat measurements (median) | 2,085 molecules |
| Removal of molecules with repeat measurement spread > 1.0 log unit | 2,044 molecules |

### 2. Chemical Space and Partitioning (Notebook 02)
- 93.4% of curated compounds pass Lipinski's Rule of Five and 94.5% satisfy Veber's rules.
- 169 molecules (8.3%) trigger a PAINS alert and were retained with flags. Alert prevalence is comparable between actives (7.7%) and inactives (9.4%), but rises among highly potent compounds (pIC50 ≥ 7.0: 11.2% vs. 6.5% for the remainder). About a third of these belong to a single azo-indole series.
- Molecules were split by Murcko scaffold into 1,681 training and 363 holdout compounds, with no shared scaffolds between sets.

### 3. QSAR Modelling and Validation (Notebooks 03–04)

#### Cross-Validation Comparison (Training Set Only, Notebook 03)

| Cross-Validation Type | R² | RMSE |
|---|---|---|
| Random Split | ~0.68 | ~0.72 |
| Scaffold Split | ~0.51 | ~0.88 |

Random cross-validation inflates performance due to close structural analogs appearing in both training and validation folds. Scaffold cross-validation provides a more realistic estimate. **Morgan fingerprints with XGBoost** achieved the highest average scaffold score (R² = 0.513, RMSE = 0.877) and was selected. Differences between feature types and models were smaller than the variation between tests, so this was a tie broken by the slightly higher average.

#### Final Holdout Evaluation (Notebook 04)

| Model | R² | RMSE | Bias |
|---|---|---|---|
| XGBoost (Morgan) | **0.435** | **0.951** | **-0.058** |
| Baseline (Mean) | -0.086 | 1.319 | — |

On unseen scaffolds, typical prediction error is approximately one pIC50 unit (~9-fold in IC50), making the model useful for relative molecule ranking rather than exact potency determination.

#### Applicability Domain (Notebook 04)

Nearest-neighbor Tanimoto similarity was computed from each holdout molecule to its closest analog in the training set (`Spearman r = -0.38` between similarity and absolute error). Prediction error is lowest when close training analogs exist (RMSE = 0.71 above 0.60 similarity) and highest for unfamiliar scaffolds (RMSE = 1.23–1.59 at or below 0.40). An applicability domain cutoff of **`0.40` Tanimoto similarity** was chosen:

| Domain Classification | Molecules | RMSE | Mean Bias (Predicted − Measured) |
|---|---|---|---|
| **Inside Domain (≥ 0.40)** | **301 (83%)** | **0.81** | **+0.06** |
| **Outside Domain (< 0.40)** | **62 (17%)** | **1.46** | **-0.61** |

Outside the applicability domain the model tends to underpredict (mean error -0.61). Several molecules with measured pIC50 of 7 to 8 are predicted at around 5.5. Because the `0.40` cutoff was selected after inspecting holdout error bins, these stratified figures characterize this holdout split rather than an external test set.

## Limitations

- ChEMBL records combine monomeric CDK2, CDK2/cyclin A, and CDK2/cyclin E biochemical assays in varying formats, introducing experimental noise. Entries with ChEMBL confidence score 8 were retained without species filtering.
- While scaffold splitting prevents exact core leakage, 65% of holdout compounds still share > 0.60 Tanimoto similarity with a training analog, so the holdout score may be slightly optimistic for new chemotypes.
- XGBoost hyperparameters were fixed without tuning, and 2D Morgan fingerprints cannot distinguish stereoisomers (139 curated molecules share connectivity with another entry).
- Outside the applicability domain, the model tends to underpredict high-potency compounds.

## Repository Structure

```text
cdk2-cheminformatics-pipeline/
├── README.md
├── environment.yml
├── data/
│   ├── raw/                 # Raw ChEMBL download
│   └── processed/           # Curated, flagged, and featured tables
├── notebooks/
│   ├── 01_data_retrieval_and_curation.ipynb
│   ├── 02_chemical_space_and_filters.ipynb
│   ├── 03_model_training.ipynb
│   └── 04_holdout_and_applicability.ipynb
├── models/                  # Serialized final model (joblib)
├── results/
│   └── tables/              # Cross-validation results, holdout results, and settings
└── figures/                 # Exported plots
```

## Setup and Usage

```bash
# Clone the repository and navigate into it
git clone https://github.com/pd-idubor/cdk2-cheminformatics-pipeline.git
cd cdk2-cheminformatics-pipeline

# Create and activate the environment
conda env create -f environment.yml
conda activate cdk2-env

# Launch Jupyter Lab
jupyter lab
```

Execute the notebooks in `notebooks/` sequentially from `01` through `04`. Random seeds (`SEED = 42`) are fixed across splits and estimators for reproducibility.

## Data Sources

- **ChEMBL:** Target [CHEMBL301](https://www.ebi.ac.uk/chembl/target_report_card/CHEMBL301/)
- **UniProt:** Entry [P24941](https://www.uniprot.org/uniprotkb/P24941) (Human CDK2)

## Author

*Precious Idubor*