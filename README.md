# Detecting Co-Travelers with Temporal Detailed People Proximity Graphs

This repository contains the code and report for the CSCI 8715 Spatial Data Science project on detecting repeated proximity and possible co-traveler patterns from location-based social network check-in data.

The project builds a Temporal Detailed People Proximity Graph, or TDPPG, from Gowalla and Brightkite check-ins. Users are treated as graph nodes, and repeated spatiotemporal encounters are converted into weighted edges. The graph is then filtered and partitioned with PyMetis to study possible user groups and repeated co-presence patterns.

## Project idea

Location-based social networks record who checked in, when they checked in, and where they checked in. These check-ins can reveal repeated proximity, but the data is sparse, noisy, and does not provide direct co-travel labels. Popular places such as malls, airports, campuses, or restaurants can create accidental co-occurrences between unrelated users.

This project handles that problem by moving from raw check-in overlaps to a weighted user-user graph. The edge weight combines repeated encounter count, encounter strength, distinct days, and shared venues. This gives more importance to repeated and meaningful proximity instead of treating every shared visit equally.

## Repository structure

```text
.
├── README.md
├── data/
│   └── README.md
├── notebooks/
│   ├── gowalla_tdppg_with_sensitivity.ipynb
│   ├── brightkite_tdppg.ipynb
│   ├── gowalla_sensitivity_only.ipynb
│   └── brightkite_sensitivity_only.ipynb
├── outputs/
│   ├── gowalla/
│   └── brightkite/
└── reports/
    └── Spatial_final_report.pdf
```

## Main notebooks

| Notebook | Purpose |
|---|---|
| `notebooks/gowalla_tdppg_with_sensitivity.ipynb` | Main Gowalla pipeline with the sensitivity analysis added at the end. This is the best notebook to keep as the primary Gowalla notebook. |
| `notebooks/brightkite_tdppg.ipynb` | Main Brightkite pipeline. |
| `notebooks/gowalla_sensitivity_only.ipynb` | Separate Gowalla sensitivity analysis notebook, kept for backup and reproducibility. |
| `notebooks/brightkite_sensitivity_only.ipynb` | Separate Brightkite sensitivity analysis notebook, kept for backup and reproducibility. |

## Method overview

The pipeline follows these steps:

1. Load Gowalla or Brightkite check-in data and social edge data.
2. Select the most active users to keep the experiment manageable.
3. Convert check-ins into venue-time events using a one-hour time bin.
4. Create valid user-pair encounters using spatial and temporal constraints.
5. Aggregate user-pair features:
   - encounter count
   - encounter strength
   - distinct days
   - shared venues
6. Build a weighted TDPPG graph.
7. Filter weak edges.
8. Partition the graph using PyMetis.
9. Evaluate the graph using ROC-AUC, PR-AUC, Precision@K, and same-partition social edge rate.
10. Run sensitivity analysis for distance threshold, time gap, and maximum event group size.

## Edge weight

The edge weight between users `i` and `j` is based on:

```text
W_ij ∝ C_ij + S_ij + D_ij + V_ij
```

where:

| Symbol | Meaning |
|---|---|
| `C_ij` | encounter count between two users |
| `S_ij` | encounter strength based on spatial and temporal closeness |
| `D_ij` | number of distinct days where the users appeared close |
| `V_ij` | number of shared venues |

A higher edge weight means stronger repeated spatiotemporal proximity.

## Final parameter settings

Based on the sensitivity analysis, the final experiments use:

| Parameter | Value |
|---|---:|
| Maximum distance | 150 meters |
| Maximum time gap | 3600 seconds |
| Minimum event size | 2 users |
| Maximum event group size | 25 users |
| Number of graph partitions | 20 |

These settings balance graph quality, noise reduction, and interpretability.

## Main results

| Dataset | ROC-AUC | PR-AUC | P@100 | P@500 | P@1000 | Same-partition social edge rate |
|---|---:|---:|---:|---:|---:|---:|
| Gowalla | 0.7588 | 0.4506 | 0.990 | 0.994 | 0.989 | 0.2037 |
| Brightkite | 0.5946 | 0.0945 | 0.440 | 0.284 | 0.248 | 0.1405 |

Gowalla produced cleaner and more reliable repeated proximity patterns. Brightkite produced a denser graph, but the ranking quality was weaker, which suggests that many of its edges are noisier co-occurrence links.

## Sensitivity analysis summary

Three parameters were tested:

| Sensitivity parameter | Tested values | Main observation |
|---|---|---|
| Maximum distance | 75 m, 150 m, 300 m | Performance stayed mostly stable for both datasets. |
| Maximum time gap | 1800 s, 3600 s, 7200 s | Larger gaps added edges but did not clearly improve ranking quality. |
| Maximum event group size | 10, 25, 50 | This had the strongest effect. Larger groups added noisy user pairs, especially for Brightkite. |

The Gowalla notebook now includes the sensitivity section at the end, so the main notebook preserves the sensitivity outputs instead of keeping them only in a separate notebook.

## Runtime summary

Pair event construction is the main bottleneck in the TDPPG pipeline. In the final report, this step took about 177.05 seconds for Gowalla and 190.51 seconds for Brightkite. PyMetis partitioning was much faster, taking about 0.14 seconds for Gowalla and 0.19 seconds for Brightkite.

## Setup

Install the main dependencies:

```bash
pip install pandas numpy matplotlib networkx scikit-learn pyarrow pymetis
```

For Google Colab, the notebooks already include an install cell:

```python
!pip -q install pymetis networkx scikit-learn pyarrow
```

## Data setup

Download the Gowalla and Brightkite datasets from SNAP and place the files in the `data/` folder.

Expected files:

```text
loc-gowalla_totalCheckins.txt.gz
loc-gowalla_edges.txt.gz
loc-brightkite_totalCheckins.txt.gz
loc-brightkite_edges.txt.gz
```

The notebooks also check common Colab paths such as `/content/` and `/mnt/data/`.

## How to run

Run the notebooks in this order:

1. `notebooks/gowalla_tdppg_with_sensitivity.ipynb`
2. `notebooks/brightkite_tdppg.ipynb`
3. Optional backup notebooks:
   - `notebooks/gowalla_sensitivity_only.ipynb`
   - `notebooks/brightkite_sensitivity_only.ipynb`

The output files are written to the `outputs/` folder. Typical outputs include:

```text
pair_events.jsonl
tdppg_pairs.csv
tdppg_partitions.csv
runtime_summary.csv
pair_eval_table.csv
```

## Limitations

The datasets do not provide direct co-travel labels, so social links are used only as an indirect validation signal. The datasets are also older, and the current version does not include full baseline comparisons against simple co-occurrence, frequency-only ranking, or distance-only methods.

## Future work

Future work can improve this project by adding stronger baselines, testing newer mobility datasets, creating synthetic ground-truth co-travel groups, improving pair-event construction with spatial indexing, and exploring hypergraph-based group modeling.

## Contributors

- Abhay Shashidhara
- Chaitanya Kadam
