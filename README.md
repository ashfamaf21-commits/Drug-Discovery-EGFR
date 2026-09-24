# EGFR Computational Drug Discovery Pipeline

An end-to-end computational workflow for identifying and prioritising potential Epidermal Growth Factor Receptor (EGFR) inhibitors, combining bioactivity data curation, cheminformatics, machine learning, virtual screening, molecular docking, protein–ligand interaction analysis and in-silico ADMET prediction.

---

## Overview

EGFR is a receptor tyrosine kinase and an established target in cancer therapy (e.g., erlotinib in non-small-cell lung cancer). This project builds a reproducible pipeline that moves from raw public bioactivity data to a short list of computationally prioritised candidate compounds.

A key focus was not only building predictive models, but also **testing how well they generalise** and being transparent about the limits of computational predictions.

---

## Target Information

| Item | Value |
|---|---|
| Target | Epidermal Growth Factor Receptor (EGFR) |
| ChEMBL ID | CHEMBL203 |
| UniProt ID | P00533 |
| Docking structure | PDB 1M17 |
| Reference ligand | Erlotinib |

---

## Workflow

```
ChEMBL EGFR bioactivity retrieval
  → Data cleaning and curation
  → IC50 to pIC50 transformation
  → Activity classification
  → Molecular descriptors and Morgan fingerprints (RDKit)
  → Machine-learning model development
  → Random train/test evaluation
  → Scaffold-based validation
  → PubChem external compound screening
  → Training-set overlap removal
  → ML-based candidate ranking
  → Tanimoto similarity analysis
  → Molecular docking (DockThor)
  → Protein–ligand interaction analysis (UCSF ChimeraX)
  → ADMET prediction (ADMETlab)
  → Final multi-parameter candidate prioritisation
```

---

## 1. Data Collection and Curation

EGFR bioactivity data were retrieved from ChEMBL using the ChEMBL Webresource Client, keeping exact IC50 measurements reported in nM.

Curation steps:

- Removal of missing SMILES and missing/invalid IC50 values
- Canonicalisation of SMILES with RDKit
- Grouping of identical structures, with duplicate IC50 values summarised by the median
- Conversion to pIC50: **pIC50 = 9 − log10(IC50 in nM)**

| Stage | Count |
|---|---|
| Raw ChEMBL records | 520 |
| Unique compounds | 387 |
| Compounds used for classification | 308 (215 active, 93 inactive) |

### Activity classification (project-defined thresholds)

- **Active:** pIC50 ≥ 6
- **Inactive:** pIC50 ≤ 5
- Intermediate compounds (5 < pIC50 < 6) were excluded to create clearer class separation

These thresholds were chosen for this project and are not universal pharmacological cut-offs.

---

## 2. Molecular Representation

- **Morgan fingerprints** (radius 2, 2,048 bits) – main representation for ML models
- **Physicochemical descriptors** – molecular weight, LogP, HBD, HBA, TPSA and rotatable bonds, used in a separate descriptor-based model

---

## 3. Machine-Learning Models

Three supervised classifiers were trained on Morgan fingerprints: **Logistic Regression, Random Forest and XGBoost** (with class weighting for imbalance). Models were evaluated using accuracy, precision, recall, F1-score, ROC-AUC, PR-AUC, confusion matrices, ROC curves and precision–recall curves.

### Random stratified split

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC | PR-AUC |
|---|---|---|---|---|---|---|
| Logistic Regression | 0.935 | 0.976 | 0.930 | 0.952 | 0.994 | 0.997 |
| Random Forest | 0.935 | 1.000 | 0.907 | 0.951 | 0.994 | 0.997 |
| **XGBoost** | **0.952** | **1.000** | **0.930** | **0.964** | **0.996** | **0.998** |

A descriptor-only Logistic Regression model reached ROC-AUC 0.927, showing that fingerprints captured substantially more activity-relevant information than simple physicochemical properties.

### Scaffold-based validation

Random splits can place structurally similar compounds in both training and test sets, which may inflate performance. A **Bemis–Murcko scaffold split** was therefore used to test generalisation to new chemical series.

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC | PR-AUC |
|---|---|---|---|---|---|---|
| XGBoost (scaffold split) | 0.951 | 1.000 | 0.750 | 0.857 | 0.849 | 0.820 |

ROC-AUC dropped from **0.996 (random split) to 0.849 (scaffold split)**. This confirms that predicting activity for structurally novel compounds is considerably harder, and that random-split results alone would overestimate real-world performance.

---

## 4. External Virtual Screening

An external library was obtained from PubChem using a structure-similarity search around erlotinib (4,317 compounds). Before prediction, SMILES were validated and canonicalised, duplicates were removed, and compounds overlapping with the training set were excluded, leaving **4,312 compounds**.

These were converted to Morgan fingerprints and ranked using the XGBoost model's active-class score. The score was used as a **prioritisation metric**, not as an experimentally measured probability of inhibition.

**Tanimoto similarity** to the training set was calculated for each candidate to judge whether top-ranked compounds were close analogues of known actives or more structurally distinct.

The top 10 candidates were shortlisted, and four were selected for structural analysis:

- CID 142677260
- CID 20747416
- CID 20747417
- CID 122232433

Selection considered ML score, structural similarity and suitability for docking.

---

## 5. Molecular Docking

Docking was performed with **DockThor** against EGFR (PDB 1M17), using erlotinib as the reference ligand and the same protocol for all compounds.

| Parameter | Value |
|---|---|
| Grid centre | X = 22.01, Y = 0.25, Z = 52.79 |
| Grid size | 20 × 20 × 20 Å |
| Evaluations | 1,000,000 |
| Population size | 750 |
| Runs | 24 |
| Soft docking | Enabled |

Erlotinib produced a DockThor score of approximately **−9.92**, and the selected candidates scored within a similar range. Docking scores were compared only within this protocol and are not experimental binding affinities.

---

## 6. Protein–Ligand Interaction Analysis

Docked complexes of erlotinib and selected candidates were examined in **UCSF ChimeraX** for hydrogen bonds, protein–ligand contacts, binding-site residues and ligand positioning in the EGFR binding pocket.

---

## 7. ADMET Prediction

In-silico ADMET properties were predicted with **ADMETlab**:

- **Physicochemical:** molecular weight, LogP, TPSA, HBD, HBA, rotatable bonds
- **Absorption and distribution:** human intestinal absorption, blood–brain barrier, P-glycoprotein inhibition and substrate status
- **Metabolism:** predicted inhibition of CYP1A2, CYP2C19, CYP2C9, CYP2D6 and CYP3A4
- **Drug-likeness and toxicity:** Ames mutagenicity, QED, Lipinski properties, synthetic accessibility

---

## 8. Final Candidate Prioritisation

Candidates were compared using multiple lines of computational evidence rather than any single metric:

- ML active-class score
- Maximum Tanimoto similarity to training compounds
- DockThor docking score and interaction energy
- Physicochemical and predicted ADMET properties
- Drug-likeness indicators

The combined results are saved in `EGFR_Final_Candidate_Comparison.csv`.

---

## Tools and Technologies

| Category | Tools |
|---|---|
| Programming and data analysis | Python, pandas, NumPy, Matplotlib |
| Machine learning | scikit-learn, XGBoost |
| Cheminformatics | RDKit |
| Databases | ChEMBL, PubChem, UniProt, Protein Data Bank |
| Molecular docking | DockThor |
| Molecular visualisation | UCSF ChimeraX |
| ADMET prediction | ADMETlab |
| Environment | Google Colab, Jupyter Notebook, GitHub |

---

## Files in This Repository

| File | Description |
|---|---|
| `EGFR_Bioactivity.ipynb` | Main notebook: data curation, ML models, validation and screening |
| `EGFR_external_predictions.csv` | Model predictions for external PubChem compounds |
| `EGFR_ranked_external_candidates.csv` | External compounds ranked by ML score |
| `EGFR_top10_candidates_for_docking.csv` | Shortlisted candidates for docking |
| `EGFR_Final_Candidate_Comparison.csv` | Final multi-parameter comparison |
| `Candidate 1.csv` – `Candidate 4.csv` | Per-candidate results |
| `Candidate_*` / `candidate *` (.zip) | Docking outputs for each candidate |
| `EGFR_Erlotinib_Redocking_.zip` | Erlotinib reference docking outputs |
| `1M17.pdb`, `1M17_cleaned.pdb`, `protein_prep.pdb` | EGFR structure files used for docking |
| `Erlotinib_EGFR_interactions.png` | Erlotinib–EGFR interaction visualisation |

---

## Limitations

- ChEMBL IC50 values come from different assays, so experimental conditions vary.
- The curated dataset is small (308 compounds), which limits model reliability.
- Activity thresholds were project-defined.
- Random-split performance is optimistic; scaffold-split performance was noticeably lower.
- The PubChem library was built by similarity to erlotinib, so it covers an erlotinib-biased chemical space.
- ML scores and docking scores are computational estimates, not experimental measurements.
- Formal crystal-pose RMSD redocking validation was not performed.
- ADMET values are predictions.
- None of the prioritised compounds has been experimentally validated for EGFR inhibition, safety or efficacy.

---

## Conclusion

This project demonstrates an integrated workflow from raw public bioactivity data to computationally prioritised EGFR candidates. The contrast between random-split and scaffold-split performance highlights the importance of realistic validation, and the final prioritisation shows the value of combining several independent computational signals rather than relying on a single score.

The selected compounds should be regarded as **computationally prioritised candidates, not validated EGFR inhibitors**; biochemical, cellular, pharmacokinetic and toxicity studies would be required for confirmation.

---

## Development Note

This project was developed as a self-directed portfolio and learning project. AI-assisted programming tools were used to help write and debug code; the workflow design, tool selection, analysis and interpretation of results were carried out by the author.

---

## Disclaimer

This project is intended for educational and research-training purposes only. Results should not be interpreted as experimental or clinical evidence.
