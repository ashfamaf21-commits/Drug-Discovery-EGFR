# EGFR Computational Drug Discovery Pipeline

## Overview

This project presents an end-to-end computational drug discovery workflow focused on the Epidermal Growth Factor Receptor (EGFR).

The project combines bioactivity data curation, cheminformatics, machine learning, virtual screening, molecular docking, protein-ligand interaction analysis, and in-silico ADMET prediction.

The main objective was to develop a reproducible workflow for identifying and prioritizing potentially EGFR-active compounds while also evaluating model generalization and the limitations of computational predictions.

## Target Information

- Target: Epidermal Growth Factor Receptor (EGFR)
- ChEMBL ID: CHEMBL203
- UniProt ID: P00533
- Docking structure: PDB 1M17
- Reference ligand: Erlotinib

## Project Workflow

ChEMBL EGFR bioactivity retrieval  
→ Data cleaning and curation  
→ IC50 to pIC50 transformation  
→ Activity classification  
→ Molecular descriptor calculation  
→ Morgan fingerprint generation  
→ Machine-learning model development  
→ Random train/test evaluation  
→ Scaffold-based validation  
→ PubChem external compound screening  
→ Training-set overlap removal  
→ ML-based candidate ranking  
→ Tanimoto similarity analysis  
→ Molecular docking against EGFR  
→ Protein-ligand interaction analysis  
→ ADMET prediction  
→ Final computational candidate prioritization  

## Bioactivity Data Collection

EGFR bioactivity data were collected from the ChEMBL database using the ChEMBL Webresource Client.

The analysis focused on:

- IC50 measurements
- exact activity relationships
- values reported in nM
- EGFR-associated compounds

The dataset was then cleaned to remove missing or invalid records.

## Data Cleaning and Curation

The preprocessing workflow included:

- removal of missing SMILES
- removal of missing or invalid IC50 values
- retention of positive IC50 measurements
- canonicalization of molecular SMILES using RDKit
- grouping of identical molecular structures
- duplicate IC50 measurements summarized using the median value

## pIC50 Transformation

IC50 values reported in nM were transformed into pIC50 values using:

pIC50 = 9 - log10(IC50 in nM)

## Activity Classification

Project-defined thresholds were used to create a binary classification task:

- Active: pIC50 >= 6
- Inactive: pIC50 <= 5
- Intermediate compounds with 5 < pIC50 < 6 were excluded

These thresholds were used for this project and should not be interpreted as universal pharmacological cut-offs.

## Molecular Descriptors

Physicochemical descriptors were calculated using RDKit.

The descriptors included:

- Molecular Weight
- LogP
- Hydrogen Bond Donors
- Hydrogen Bond Acceptors
- Topological Polar Surface Area
- Rotatable Bonds

A descriptor-based Logistic Regression model was also used to explore associations between physicochemical properties and EGFR activity.

## Molecular Fingerprints

Morgan fingerprints were generated using RDKit with:

- Radius: 2
- Fingerprint size: 2048 bits

These fingerprints were used as the main molecular representation for the machine-learning models.

## Machine Learning Models

Three supervised classification algorithms were evaluated:

- Logistic Regression
- Random Forest
- XGBoost

The models were evaluated using:

- Accuracy
- Precision
- Recall
- F1-score
- ROC-AUC
- PR-AUC
- Confusion Matrix
- ROC Curve
- Precision-Recall Curve

## Random Train/Test Evaluation

The curated dataset was divided into training and test sets using stratified random splitting.

The models achieved high predictive performance on the random test set.

However, random splitting can place structurally similar compounds in both the training and test sets, potentially leading to optimistic performance estimates.

## Scaffold-Based Validation

A scaffold-based split was also performed to provide a more challenging evaluation of model generalization.

Performance was lower than with the random split, showing that prediction of structurally different compounds is more difficult than prediction of compounds similar to the training data.

This step was included to assess the ability of the model to generalize to new chemical scaffolds.

## External Compound Screening

An external compound library was obtained from PubChem using an erlotinib similarity search.

Before prediction:

- molecular SMILES were validated
- molecules were canonicalized
- duplicate structures were removed
- exact overlaps with the training dataset were identified
- training-set overlaps were removed

The remaining compounds were converted into Morgan fingerprints and screened using the trained XGBoost model.

## Candidate Ranking

External compounds were ranked using the XGBoost model-estimated active-class score.

The model score was used as a prioritization metric and should not be interpreted as an experimentally measured probability of EGFR inhibition.

## Tanimoto Similarity Analysis

Tanimoto similarity was calculated between external candidates and compounds in the training dataset.

For each candidate, the maximum similarity to the training set was calculated.

This was used to assess whether highly ranked compounds were close analogues of existing training compounds or represented somewhat different molecular structures.

## Selected Candidates

Four high-priority compounds were selected for further structural analysis:

- CID 142677260
- CID 20747416
- CID 20747417
- CID 122232433

Candidate selection considered machine-learning score, structural similarity, and suitability for downstream docking analysis.

## Molecular Docking

Molecular docking was performed using DockThor.

### Receptor

- EGFR
- PDB ID: 1M17

### Reference Ligand

- Erlotinib

### Binding Site

Approximate docking center:

- X = 22.01
- Y = 0.25
- Z = 52.79

Grid size:

- 20 × 20 × 20 Å

### Docking Parameters

- Evaluations: 1,000,000
- Population size: 750
- Runs: 24
- Soft docking: enabled

The same docking protocol was applied to the selected candidates and the erlotinib reference.

## Docking Results

Erlotinib reference docking produced a DockThor score of approximately:

-9.922

The selected candidates produced docking scores within a similar computational range.

Docking scores were interpreted comparatively within the same protocol and should not be treated as experimental binding affinities.

## Protein-Ligand Interaction Analysis

Selected docked complexes were analyzed using UCSF ChimeraX.

The analysis focused on:

- hydrogen bonds
- protein-ligand contacts
- binding-site residues
- ligand positioning within the EGFR binding pocket

Interaction analysis was performed for erlotinib and selected prioritized candidates.

## ADMET Prediction

In-silico ADMET prediction was carried out using ADMETlab.

Properties evaluated included:

### Physicochemical Properties

- Molecular Weight
- LogP
- TPSA
- Hydrogen Bond Donors
- Hydrogen Bond Acceptors
- Rotatable Bonds

### Absorption and Distribution

- Human intestinal absorption
- Blood-brain barrier prediction
- P-glycoprotein inhibition
- P-glycoprotein substrate prediction

### Metabolism

Predicted inhibition of:

- CYP1A2
- CYP2C19
- CYP2C9
- CYP2D6
- CYP3A4

### Drug-Likeness and Toxicity

- Ames mutagenicity prediction
- QED
- Lipinski properties
- Synthetic accessibility

These values are computational predictions and require experimental confirmation.

## Final Candidate Comparison

The final candidate comparison integrated multiple computational evidence sources:

- machine-learning active-class score
- maximum Tanimoto similarity to training compounds
- DockThor docking score
- interaction energy
- physicochemical properties
- predicted ADMET properties
- drug-likeness indicators

Final outputs included:

- EGFR_Final_Candidate_Comparison.csv
- EGFR_Final_Summary.csv

## Skills Demonstrated

### Machine Learning

- supervised binary classification
- Logistic Regression
- Random Forest
- XGBoost
- model evaluation
- ROC-AUC analysis
- PR-AUC analysis
- confusion matrix analysis
- random validation
- scaffold-based validation

### Cheminformatics

- SMILES processing
- molecular canonicalization
- molecular descriptors
- Morgan fingerprints
- Tanimoto similarity
- molecular structure comparison

### Computational Drug Discovery

- bioactivity data curation
- virtual screening
- candidate prioritization
- molecular docking
- protein-ligand interaction analysis
- ADMET prediction
- multi-parameter candidate assessment

### Data Science

- data cleaning
- feature engineering
- exploratory analysis
- model comparison
- data visualization
- integration of outputs from multiple computational tools

## Tools and Technologies

### Programming and Data Analysis

- Python
- pandas
- NumPy
- matplotlib

### Machine Learning

- scikit-learn
- XGBoost

### Cheminformatics

- RDKit

### Databases

- ChEMBL
- PubChem
- UniProt
- Protein Data Bank

### Molecular Docking

- DockThor

### Molecular Visualization

- UCSF ChimeraX

### ADMET Prediction

- ADMETlab

### Development Environment

- Google Colab
- Jupyter Notebook
- GitHub

## Limitations

- ChEMBL IC50 values were obtained from different assays, so experimental conditions may vary.
- Only exact IC50 measurements reported in nM were retained.
- Duplicate measurements were summarized using the median IC50.
- Activity thresholds used in the project were project-defined.
- Random train/test splitting may overestimate performance when structurally similar compounds occur in both datasets.
- Scaffold-based performance was lower, indicating reduced generalization to new structural series.
- The PubChem external library was generated using similarity to erlotinib and therefore represents an erlotinib-biased chemical space.
- Machine-learning active-class scores are computational outputs and not experimentally measured probabilities.
- Molecular docking scores are approximate computational scoring values.
- Formal crystal-pose RMSD redocking validation was not performed.
- ADMETlab results are computational predictions.
- The prioritized compounds have not been experimentally validated for EGFR inhibition, safety, or therapeutic efficacy.

## Conclusion

This project demonstrates an integrated computational workflow for EGFR-focused drug discovery using machine learning, cheminformatics, molecular docking, protein-ligand interaction analysis, and ADMET prediction.

The workflow progressed from raw public bioactivity data through molecular representation, predictive modelling, scaffold-based validation, external screening, structural analysis, and final candidate prioritization.

The project also demonstrates the importance of combining multiple computational evidence sources rather than relying only on a single metric such as machine-learning score or docking score.

The final compounds should therefore be considered computationally prioritized candidates rather than experimentally validated EGFR inhibitors.

Further biochemical, cellular, pharmacokinetic, and toxicity studies would be required for experimental validation.

## Project Purpose

This project was developed as a portfolio and learning project to build practical skills at the intersection of:

- Artificial Intelligence
- Bioinformatics
- Cheminformatics
- Molecular Biology
- Computational Chemistry
- Drug Discovery
