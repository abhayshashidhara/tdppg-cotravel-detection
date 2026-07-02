# Detecting Co-Travelers with Temporal Detailed People Proximity Graphs

This repository contains the code and report for the CSCI 8715 Spatial Data Science project on detecting repeated proximity and possible co-traveler patterns from location-based social network check-in data.

The project builds a Temporal Detailed People Proximity Graph, or TDPPG, from Gowalla and Brightkite check-ins. Users are represented as graph nodes, and repeated spatiotemporal encounters are converted into weighted user-user edges. The graph is then filtered and partitioned using PyMetis to study repeated co-presence patterns and possible user groups.

## Project idea

Location-based social networks record who checked in, when they checked in, and where they checked in. These check-ins can reveal repeated proximity between users, but the data is sparse, noisy, and does not provide direct co-travel labels.

A major challenge is that popular places such as malls, airports, campuses, restaurants, or public venues can create accidental co-occurrences between unrelated users. This project handles that problem by moving from raw check-in overlaps to a weighted user-user graph.

The edge weight combines repeated encounter count, encounter strength, distinct days, and shared venues. This gives more importance to repeated and meaningful proximity instead of treating every shared visit equally.

## Repository structure

```text
.
├── README.md
├── notebooks/
│   ├── gowalla_tdppg_with_sensitivity.ipynb
│   ├── brightkite_tdppg.ipynb
│   ├── gowalla_sensitivity_only.ipynb
│   └── brightkite_sensitivity_only.ipynb
└── reports/
    └── Spatial_final_report.pdf

The raw datasets and generated outputs are not committed to this repository. They can be downloaded and created locally when running the notebooks.

Main notebooks
Notebook	Purpose
notebooks/gowalla_tdppg_with_sensitivity.ipynb	Main Gowalla pipeline with sensitivity analysis included at the end. This is the primary Gowalla notebook.
notebooks/brightkite_tdppg.ipynb	Main Brightkite pipeline for TDPPG construction, graph partitioning, and evaluation.
notebooks/gowalla_sensitivity_only.ipynb	Separate Gowalla sensitivity analysis notebook, kept for backup and reproducibility.
notebooks/brightkite_sensitivity_only.ipynb	Separate Brightkite sensitivity analysis notebook, kept for backup and reproducibility.
Report

The final project report is available in:

reports/Spatial_final_report.pdf

The report contains the project motivation, methodology, implementation details, results, sensitivity analysis, runtime discussion, limitations, and future work.

Method overview

The pipeline follows these steps:

Load Gowalla or Brightkite check-in data and social edge data.
Select the most active users to keep the experiment computationally manageable.
Convert check-ins into venue-time events using a one-hour time bin.
Create valid user-pair encounters using spatial and temporal constraints.
Aggregate user-pair features:
encounter count
encounter strength
distinct days
shared venues
Build a weighted Temporal Detailed People Proximity Graph.
Filter weak edges from the graph.
Partition the graph using PyMetis.
Evaluate the graph using ROC-AUC, PR-AUC, Precision@K, and same-partition social edge rate.
Run sensitivity analysis for distance threshold, time gap, and maximum event group size.
Edge weight

The edge weight between users i and j is based on:

W_ij ∝ C_ij + S_ij + D_ij + V_ij

where:

Symbol	Meaning
C_ij	Encounter count between two users
S_ij	Encounter strength based on spatial and temporal closeness
D_ij	Number of distinct days where the users appeared close
V_ij	Number of shared venues

A higher edge weight means stronger repeated spatiotemporal proximity between two users.

Final parameter settings

Based on the sensitivity analysis, the final experiments use:

Parameter	Value
Maximum distance	150 meters
Maximum time gap	3600 seconds
Minimum event size	2 users
Maximum event group size	25 users
Number of graph partitions	20

These settings balance graph quality, noise reduction, runtime, and interpretability.

Main results
Dataset	ROC-AUC	PR-AUC	P@100	P@500	P@1000	Same-partition social edge rate
Gowalla	0.7588	0.4506	0.990	0.994	0.989	0.2037
Brightkite	0.5946	0.0945	0.440	0.284	0.248	0.1405

Gowalla produced cleaner and more reliable repeated proximity patterns. Brightkite produced a denser graph, but the ranking quality was weaker, which suggests that many Brightkite edges were noisier co-occurrence links.

Sensitivity analysis summary

Three parameters were tested:

Sensitivity parameter	Tested values	Main observation
Maximum distance	75 m, 150 m, 300 m	Performance stayed mostly stable for both datasets.
Maximum time gap	1800 s, 3600 s, 7200 s	Larger time gaps added more edges but did not clearly improve ranking quality.
Maximum event group size	10, 25, 50	This had the strongest effect. Larger groups added noisy user pairs, especially for Brightkite.

The main Gowalla notebook includes the sensitivity analysis at the end, so the primary pipeline preserves the sensitivity outputs directly. Separate sensitivity-only notebooks are also included for backup and reproducibility.

Runtime summary

Pair-event construction is the main bottleneck in the TDPPG pipeline. In the final report, this step took about 177.05 seconds for Gowalla and 190.51 seconds for Brightkite.

PyMetis partitioning was much faster, taking about 0.14 seconds for Gowalla and 0.19 seconds for Brightkite.

Setup

Install the main dependencies:

pip install pandas numpy matplotlib networkx scikit-learn pyarrow pymetis

For Google Colab, the notebooks already include an install cell:

!pip -q install pymetis networkx scikit-learn pyarrow
Data setup

The raw Gowalla and Brightkite datasets are not included in this repository because the files are large.

Download the datasets from Stanford SNAP and place them locally in a data/ folder before running the notebooks.

Expected local files:

data/
├── loc-gowalla_totalCheckins.txt.gz
├── loc-gowalla_edges.txt.gz
├── loc-brightkite_totalCheckins.txt.gz
└── loc-brightkite_edges.txt.gz

The notebooks also check common local and Colab paths such as:

data/
/content/
/content/data/
/mnt/data/
/mnt/data/data/
How to run

Run the notebooks in this order:

notebooks/gowalla_tdppg_with_sensitivity.ipynb
notebooks/brightkite_tdppg.ipynb

Optional backup notebooks:

notebooks/gowalla_sensitivity_only.ipynb
notebooks/brightkite_sensitivity_only.ipynb
Expected outputs

The notebooks generate output files locally while running. These files are not committed to the repository.

Expected output structure:

outputs/
├── gowalla/
│   ├── pair_events.jsonl
│   ├── tdppg_pairs.csv
│   ├── tdppg_partitions.csv
│   ├── runtime_summary.csv
│   ├── pair_eval_table.csv
│   └── sensitivity_summary.csv
└── brightkite/
    ├── pair_events.jsonl
    ├── tdppg_pairs.csv
    ├── tdppg_partitions.csv
    ├── runtime_summary.csv
    ├── pair_eval_table.csv
    └── sensitivity_summary.csv
File	Description
pair_events.jsonl	Raw valid user-pair encounter events created using spatial and temporal constraints.
tdppg_pairs.csv	Aggregated user-pair features and final TDPPG edge weights.
tdppg_partitions.csv	User-to-partition mapping generated using PyMetis.
runtime_summary.csv	Runtime summary for pair-event construction and graph partitioning.
pair_eval_table.csv	Main evaluation results including ROC-AUC, PR-AUC, Precision@K, and partition quality.
sensitivity_summary.csv	Sensitivity analysis results for distance threshold, time gap, and event group size.
Recommended .gitignore

The following files and folders should be kept out of GitHub:

data/
outputs/
*.gz
*.csv
*.jsonl
.ipynb_checkpoints/
__pycache__/

This keeps the repository clean while still allowing the project to be reproduced locally.

Limitations

The datasets do not provide direct co-travel labels, so social links are used only as an indirect validation signal. The datasets are also older, and the current version does not include full baseline comparisons against simple co-occurrence, frequency-only ranking, or distance-only methods.

Future work

Future improvements can include stronger baseline comparisons, newer mobility datasets, synthetic ground-truth co-travel groups, spatial indexing for faster pair-event construction, and hypergraph-based group modeling.

Contributors
Abhay Shashidhara
Chaitanya Kadam
