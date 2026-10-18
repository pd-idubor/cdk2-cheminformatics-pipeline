# CDK2 Cheminformatics Workflow

A predictive cheminformatics pipeline for Cyclin-dependent kinase 2 (CDK2) featuring automated ChEMBL data curation, molecular standardization, scaffold-aware machine learning modeling, and validated molecular docking.

> **Status:** work in progress.

---

## Project Specifications

| Item | Detail |
|---|---|
| Target | Cyclin-dependent kinase 2 (*Homo sapiens*) |
| UniProt | [P24941](https://www.uniprot.org/uniprotkb/P24941) |
| ChEMBL target | [CHEMBL301](https://www.ebi.ac.uk/chembl/target_report_card/CHEMBL301/) |
| Data Endpoint | $IC_{50}$ (nM, standard relation "="), converted to $pIC_{50}$ (M) |


## Pipeline Overview

1. **Data curation:** programmatic retrieval of CDK2 $IC_{50}$ data from ChEMBL, molecule standardization, duplicate aggregation, and a logged record of every filtering step.
2. **Chemical space and SAR analysis:** PAINS flags, Lipinski/Veber properties, Murcko scaffolds, clustering, MCS.
3. **QSAR modelling**
4. **Virtual screening**
5. **Docking**


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

## Author

_Precious Idubor_