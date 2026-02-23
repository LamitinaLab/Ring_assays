# C. elegans Osmotic Ring Assay Analysis

This project provides a reproducible pipeline to analyze C. elegans osmotic ring assays from WormLab exports. The goal is to transform raw WormLab files into a standardized set of Parquet files and generate key analysis plots.

## Project Structure

```
wormlab-ring/
├── data/
│   ├── raw/
│   │   └── <dataset_id>/
│   │       ├── Position.csv
│   │       ├── Measurements.csv
│   │       └── labels/
│   ├── processed/
│   │   ├── dataset_index.parquet
│   │   ├── geometry.parquet
│   │   └── tracks.parquet
│   └── figures/
├── notebooks/
│   ├── 00_build_dataset_index.ipynb
│   ├── 01_ingest_to_parquet.ipynb
│   └── 02_analysis.ipynb
└── README.md
```

## Pipeline Overview

1. **`00_build_dataset_index.ipynb`**: Scans `data/raw` to build a dataset index and saves `dataset_index.parquet` in `data/processed`.
2. **`01_ingest_to_parquet.ipynb`**: Ingests WormLab exports per dataset (`Position.csv`, `Measurements.csv`, labels), producing `geometry.parquet` (ring boundaries, plate geometry) and `tracks.parquet` (per-detection kinematics).
3. **`02_analysis.ipynb`**: Runs tracklet-based analyses and figures using the processed Parquet files. Key sections include:
	- Speed vs Time (track-weighted with SEM)
	- Ring Occupancy vs Time (fraction inside, detection and track counts)
	- Track + Geometry visualization per dataset
	- Distance Traveled and Mean Speed while in the ring (dataset-specific inner/outer bounds)
	- Resolved Cross Rate (Cross / (Cross + Abort)) per dataset and per genotype
	- Time to First Crossing (by boundary), per-genotype summaries
	- Statistical comparisons (Mann–Whitney for two groups, Kruskal–Wallis for ≥3)

## Design Rules

- `track_uid` is unique per track, derived from `dataset_id` and the source track ID.
- Speed calculations are gap-aware (no velocity across missing frames).
- Aggregations specify weighting (detection-, track-, or plate-weighted) explicitly.
- Distances to the ring use the dataset-specific midline: average of inner/outer ring radii and centers; signed distance `dist_to_ring_um` is computed early and reused.
- Inner/outer ring bounds are derived per dataset; labels swapped if inner > outer.
- Entry detection uses a threshold crossing at the inner boundary (boolean edge), with a fallback to first threshold reach.
- Statistical tests use nonparametrics (`scipy.stats`), with printed significance markers.

## Environment Setup

Activate the project virtual environment in the repo root:

```bash
source .venv/bin/activate
```

Recommended packages (installed in the venv): `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `pyarrow`.

## Running the Notebooks

1. Run the notebooks in order: 1) dataset index, 2) ingestion, 3) analysis.
2. In the analysis notebook, ensure the early setup cells are executed (imports, data loads, and computation of `dist_to_ring_um`).
3. Analyses recompute dataset-specific ring bounds from `geometry.parquet` to avoid stale globals.
4. Outputs are written to `data/figures` and CSV summaries to `data/figures`.

## Outputs

Representative outputs saved by `02_analysis.ipynb`:

- [data/figures/speed_vs_time_by_genotype_with_sem.png](data/figures/speed_vs_time_by_genotype_with_sem.png)
- [data/figures/ring_occupancy_and_counts_vs_time_by_genotype.png](data/figures/ring_occupancy_and_counts_vs_time_by_genotype.png)
- [data/figures/ring_transit_distance_speed.png](data/figures/ring_transit_distance_speed.png)
- [data/figures/time_to_first_crossing_by_boundary.png](data/figures/time_to_first_crossing_by_boundary.png)
- [data/figures/resolved_cross_rate_by_genotype.png](data/figures/resolved_cross_rate_by_genotype.png)
- [data/figures/ring_transit_data.csv](data/figures/ring_transit_data.csv)
- [data/figures/ring_transit_summary.csv](data/figures/ring_transit_summary.csv)
- [data/figures/ring_engagement_outcomes.csv](data/figures/ring_engagement_outcomes.csv)
- [data/figures/ring_engagement_dataset_summary.csv](data/figures/ring_engagement_dataset_summary.csv)
- [data/figures/ring_engagement_genotype_summary.csv](data/figures/ring_engagement_genotype_summary.csv)
