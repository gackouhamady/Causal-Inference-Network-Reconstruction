# Network Reconstruction & Causal Inference Projects 

This repository contains a collection of advanced projects focusing on **Network Reconstruction** and **Bayesian Networks**, applied to both synthetic benchmarks and real-world biological data.

**Context:** Master in Machine Learning for Data Science (University Paris Cité).

##  Tech Stack
* **Language:** R (Jupyter Notebooks)
* **Libraries:** `bnlearn`, `miic`, `igraph`, `pcalg`
* **Concepts:** Causal Discovery, Directed Acyclic Graphs (DAGs), Conditional Independence, Information Theory.

## Project Overview

### 1. Gene Regulatory Network Analysis (TP1)
* **Objective:** Reconstruction of hematopoietic and endothelial regulatory networks.
* **Methods:** Comparison between Correlation networks and Partial Correlation networks.
* **Key Findings:** Demonstrated that partial correlation recovers direct TF interactions better than simple correlation, aligning with the reference network (Verny et al.).

### 2. Benchmarking Bayesian Network Learners (TP2)
* **Objective:** Recovering the structure of the "Insurance" Bayesian Network (categorical data).
* **Algorithms:** Hill-Climbing (Score-based), PC (Constraint-based), ARACNE (Pairwise MI), and MIIC.
* **Results:** * *Hill-Climbing* tends to overfit.
    * *MIIC* proved to be the most robust approach for recovering global structure.
    * Implemented rigorous evaluation metrics (False Positives/Negatives, Precision/Recall).

### 3. Genomic Alterations Network (TP3)
* **Objective:** Inferring dependencies between genomic alterations in breast cancer data.
* **Challenge:** Handling mixed mutation/expression variables and high dimensionality.
* **Outcome:** Identified key hubs and recurrent modules despite data noise. Highlights the trade-off between algorithm sensitivity and biological interpretability.








---
*Author: Hamady Gackou*
