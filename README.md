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
================================================================================
>>> INITIALIZING GRN INFERENCE PIPELINE...
>>> SYSTEM: R_Analysis_Environment
>>> MODE: COMPARATIVE (CORRELATION vs PARTIAL CORRELATION)
================================================================================

################################################################################
# PHASE 1: DATA INPUT & PREPROCESSING
################################################################################
[INPUT]     Expression Matrix (X)
[DIMENSION] 3934 Cells (n) x 33 TFs (p)
[TYPE]      Binary/Continuous (Ct values converted to 0/1 or intensities)

    +-------+-------+-------+      +-------+
    | Gene1 | Gene2 | ...   |      |   X   |
    +-------+-------+-------+      +-------+
    |   1   |   0   | ...   |  ->  | Raw   |
    |   0   |   1   | ...   |      | Data  |
    |   1   |   1   | ...   |      +-------+
    +-------+-------+-------+

[CRITICAL ANALYSIS]
> Why?      To organize measurements into a computable algebraic structure.
> Risk:     Single-cell data is noisy. "0" can mean "gene off" or "technical dropout."
            If not normalized (log-transform/scaling), high-expression genes will 
            dominate the variance, skewing correlations.

--------------------------------------------------------------------------------

################################################################################
# PHASE 2: DEPENDENCY CALCULATION (THE FORK)
################################################################################

>>> OPTION A: STANDARD CORRELATION (PEARSON)
---------------------------------------------------------------------------
[CONCEPT]   Measures "Co-expression". How much do Gene A and Gene B vary together?
            It captures DIRECT + INDIRECT relationships.

[FORMULA]   Pearson Coefficient (r):
            
            r(A,B) = cov(A,B) / (sigma_A * sigma_B)

            Where:
            cov   = Covariance
            sigma = Standard Deviation

[VISUALIZATION - THE "TRIANGLE" PROBLEM]
    
    Reality:      Gene C activates A AND B. (A and B do not touch).
    
          C
         / \      (C regulates both)
        v   v
        A   B

    Inferred by Correlation:
    
          C
         / \
        A---B     <-- FALSE LINK! 
                      Correlation sees A and B rising together 
                      and assumes a connection.

[CRITICAL ANALYSIS]
> Pros:     Computationally cheap. Robust for small datasets. 
            Great for identifying global "Modules" (groups of genes working together).
> Cons:     Cannot distinguish causation. High False Positive rate (confounding factors).
            Resulting graphs are often "hairballs" (too dense).

---------------------------------------------------------------------------
>>> OPTION B: PARTIAL CORRELATION (GGM / GLASSO)
---------------------------------------------------------------------------
[CONCEPT]   Measures "Direct Interaction". Association between A and B, 
            controlling for the influence of ALL other genes (C, D, E...).

[FORMULA]   Based on the Precision Matrix (Omega):
            
            1. Calculate Covariance Matrix (Sigma).
            2. Invert it: Omega = Sigma^(-1)
            3. Partial Corr (rho_ij):
            
            rho_ij = - (omega_ij) / sqrt(omega_ii * omega_jj)

[VISUALIZATION - RESOLVING THE TRIANGLE]

    Reality:      Gene C activates A AND B.
    
          C
         / \
        v   v
        A   B

    Inferred by Partial Correlation:
    
          C
         / \
        A   B     <-- CORRECT! 
                      Mathematically removes the effect of C.
                      The link A-B disappears.

[CRITICAL ANALYSIS]
> Pros:     Removes indirect effects. Much closer to a "mechanistic" regulatory map.
            Sparse output (cleaner graph).
> Cons:     Requires n >> p (sample size must be larger than gene count) to invert matrix.
            Sensitive to noise. If "C" is not in your dataset, you cannot control for it.

################################################################################
# PHASE 3: ADJACENCY MATRIX GENERATION (THRESHOLDING)
################################################################################
[INPUT]     Similarity Matrix (33x33) from Phase 2.
[ACTION]    Apply Cut-off (tau).

[LOGIC]
    IF |score| > threshold THEN edge = 1
    ELSE edge = 0

[VISUALIZATION]

   Correlation Matrix         Adjacency Matrix (Binary)
   +-----+-----+-----+        +-----+-----+-----+
   | 1.0 | 0.8 | 0.1 |  >0.5  |  1  |  1  |  0  |
   | 0.8 | 1.0 | 0.2 |  --->  |  1  |  1  |  0  |
   | 0.1 | 0.2 | 1.0 |        |  0  |  0  |  1  |
   +-----+-----+-----+        +-----+-----+-----+

[CRITICAL ANALYSIS]
> Why?      To remove weak associations that are likely just biological noise.
> Risk:     Selection bias. 
            - Threshold 0.3: Graph is too dense (many false positives).
            - Threshold 0.7: Graph is disconnected (might lose real but weak interactions).
            Biological reality does not have a hard "cutoff," this is a mathematical simplification.

################################################################################
# PHASE 4: FINAL GRAPH CONSTRUCTION
################################################################################
[INPUT]     Adjacency Matrix.
[OUTPUT]    Network Graph object (Nodes & Edges).

[VISUALIZATION - LAYOUT ALGORITHM]

      (Nodes repel like magnets)
           O         O
            \       /
             \     /   (Edges act like springs)
              \   /
                O

[FINAL COMPARISON SUMMARY]

    | FEATURE          | CORRELATION GRAPH           | PARTIAL CORR GRAPH          |
    |------------------|-----------------------------|-----------------------------|
    | Edge Meaning     | Co-expression               | Conditional Dependence      |
    | Biology          | Functional Modules          | Direct Regulatory Paths     |
    | Density          | High (Dense/Clustered)      | Low (Sparse)                |
    | Indirect Links   | Included (Confounded)       | Removed (mostly)            |
    | Lineage Modules  | Clearly visible (Red/Blue)  | Less obvious, more fragmented|

================================================================================
>>> PIPELINE COMPLETE.
>>> READY FOR VISUALIZATION.
================================================================================

















---
*Author: Hamady Gackou*
