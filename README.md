# Variable Star Discovery and Classification Pipeline

## Overview

This repository contains the complete computational pipeline for MSc thesis:

> *Variable Star Discovery and Classification Using TESS Photometry and Machine Learning*

The project consists of three end-to-end Python pipelines that work together to discover new variable stars in unexplored open clusters and classify them using machine learning.

---

## Repository Structure

```
variable-star-pipeline/
│
├── pipelines/
│   ├── variable_star_discovery_pipeline.py   # Pipeline A — Discovery (known/benchmark fields)
│   ├── unexplored_cluster_pipeline.py        # Pipeline A — Discovery (UPK unexplored clusters)
│   └── known_variable_classifier.py          # Pipeline B — Supervised ML classification
│
├── presentation/
│   ├── thesis_presentation.tex               # Pre-defence Beamer presentation (LaTeX source)
│   └── thesis_presentation.pdf               # Compiled PDF
│
├── proposal/
│   ├── thesis_proposal.tex                   # Thesis proposal (LaTeX source)
│   └── thesis_proposal.pdf                   # Compiled PDF
│
├── results/                                  # Output directory (generated at runtime)
│   └── .gitkeep
│
├── requirements.txt                          # Python dependencies
├── .gitignore
└── README.md
```

---

## Pipelines

### Pipeline A1 — Variable Star Discovery (`variable_star_discovery_pipeline.py`)

Discovers new variable stars in a user-specified sky field (e.g., an open cluster) using TESS photometry.

**Workflow:**
1. Resolves target field via SIMBAD
2. Queries TIC (TESS Input Catalog) for stars in the field
3. Downloads TESS light curves via `lightkurve` (SPOC → QLP → TGLC fallback)
4. Computes variability statistics (σ, IQR, amplitude, Stetson-J)
5. Runs Lomb-Scargle period search (0.05–13 days, FAP threshold 0.05)
6. Cross-matches candidates against AAVSO VSX within 21 arcsec
7. Flags stars with no VSX match as new discovery candidates
8. Outputs PDF report + CSV catalogs

**Usage:**
```bash
python pipelines/variable_star_discovery_pipeline.py
```

Edit the `CONFIG` block at the top to change the target cluster, magnitude limits, and thresholds.

**Key config options:**
```python
"cluster_name"    : "NGC 2516",   # or set to None and use ra_deg/dec_deg
"tmag_max"        : 14.5,
"ls_fap_thresh"   : 0.05,
"vsx_match_radius_arcsec": 21.0,
```

---

### Pipeline A2 — Unexplored UPK Cluster Discovery (`unexplored_cluster_pipeline.py`)

Automatically screens all 60 UPK clusters (Sim et al. 2019) for prior variable-star publications and runs discovery on unexplored targets.

**Workflow:**
- **Phase 0:** Queries NASA ADS API per cluster for variable-star papers; checks TESS sector coverage. Only clusters with 0 papers proceed.
- **Phase 1:** Runs the full discovery pipeline (same as A1) on each unexplored cluster using parallel downloads (ThreadPoolExecutor, 3 workers) and disk caching.

**Usage:**
```bash
python pipelines/unexplored_cluster_pipeline.py
```

**Outputs** (in `upk_discovery_results/`):
- `cluster_screening_results.csv` — all 60 clusters with ADS paper count and TESS coverage
- `UPK_XX_report.pdf` — per-cluster diagnostic report
- `master_discovery_catalog.csv` — all new candidates across clusters
- `MASTER_DISCOVERY_REPORT.pdf` — combined summary PDF

---

### Pipeline B — Supervised ML Classification (`known_variable_classifier.py`)

Trains and evaluates three classifiers (Random Forest, Gradient Boosting, SVM) on known variable stars from AAVSO VSX to classify variable type.

**Features extracted (19 total):**

| Category | Features |
|---|---|
| Periodic | log₁₀(P), LS power, log(FAP), Fourier amplitude, R₂₁, φ₂₁, R₃₁, φ₃₁, R² |
| Statistical | mean, σ, skewness, kurtosis, IQR, MAD, raw amplitude, Stetson-J, von Neumann η, beyond-1σ fraction |

**Target classes:** EW, EA, EB, RRAB, RRC, DSCT, SR, M

**Usage:**
```bash
python pipelines/known_variable_classifier.py
```

**Outputs** (in `classifier_results/`):
- `classification_report.pdf` — confusion matrices, feature importance, t-SNE projection
- `feature_table.csv` — extracted features for all training stars
- `test_predictions.csv` — true vs predicted labels
- `random_forest_model.joblib` — trained RF pipeline (reload with `joblib.load(...)`)

---

## Installation

```bash
# Clone the repository
git clone https://github.com/adrita-khan/variable-star-pipeline.git
cd variable-star-pipeline

# Install dependencies
pip install -r requirements.txt
```

**Python version:** 3.10 or higher recommended.

---

## Data Sources

All data used in this project is publicly available:

| Source | Access | Use |
|---|---|---|
| TESS SPOC/QLP light curves | `lightkurve` / MAST | All pipelines |
| TESS Input Catalog (TIC) | `astroquery.mast` | Star selection |
| AAVSO VSX | `astroquery.vizier` (B/vsx/vsx) | Training labels, cross-match |
| Gaia DR3 | `astroquery.gaia` | Membership (future extension) |
| UPK catalogue | Sim et al. 2019 | Target cluster list |
| NASA ADS | Public API | Literature screening |
| SIMBAD | `astroquery.simbad` | Object resolution |

No proprietary data or telescope time required.

---

## Expected Outputs

| Pipeline | Expected yield |
|---|---|
| Discovery (per cluster) | 3–15 new variable candidates |
| Supervised classifier | Macro-F1 ≥ 0.85 across 8 classes |
| t-SNE + HDBSCAN | Cluster structure in 19-D feature space |


---

## References

- Sim et al. (2019) — UPK catalogue. *ApJL* 880, L25.
- Richards et al. (2011) — Random Forest for variable stars. *ApJ* 733, 10.
- Gao et al. (2025) — TESS periodic variable classification. *ApJS* 276, 57.
- Chen et al. (2020) — ZTF periodic variable catalogue. *ApJS* 249, 18.
- Kim & Bailer-Jones (2016) — Fourier + statistical features. *A&A* 587, A18.
- Fetherolf et al. (2023) — TESS prime mission variability. *ApJS* 268, 4.
- Stetson (1996) — Variability indices. *PASP* 108, 851.
- Zechmeister & Kürster (2009) — Generalised Lomb-Scargle. *A&A* 496, 577.

---

## Contact

**Adrita Khan**  
adrita.khan.251@northsouth.edu  
North South University, Dhaka, Bangladesh
