# Quick Start Guide

## 1. Install dependencies

```bash
pip install -r requirements.txt
```

---

## 2. Run Pipeline A1 — Discovery on a known cluster (benchmark)

```bash
python pipelines/variable_star_discovery_pipeline.py
```

Default target: **NGC 2516** (0 new expected — used to validate the pipeline).  
Change target in the `CONFIG` block at the top of the file:

```python
"cluster_name" : "NGC 2516",   # ← change this
```

**Outputs:**
```
discovery_report.pdf
new_variable_candidates.csv
all_stars_variability.csv
```

---

## 3. Run Pipeline A2 — Discovery on unexplored UPK clusters

```bash
python pipelines/unexplored_cluster_pipeline.py
```

This will:
1. Screen 60 UPK clusters via NASA ADS (~5 minutes)
2. Run full photometric analysis on the first 5 unexplored clusters found

**Outputs in `upk_discovery_results/`:**
```
cluster_screening_results.csv
UPK_XX_report.pdf          (one per cluster analysed)
master_discovery_catalog.csv
MASTER_DISCOVERY_REPORT.pdf
```

---

## 4. Run Pipeline B — Train the ML classifier

```bash
python pipelines/known_variable_classifier.py
```

Queries AAVSO VSX for labelled stars, downloads TESS light curves,
trains Random Forest / Gradient Boosting / SVM, and evaluates all three.

**Outputs in `classifier_results/`:**
```
classification_report.pdf
feature_table.csv
test_predictions.csv
random_forest_model.joblib
```

**Reload the trained model later:**
```python
import joblib
bundle = joblib.load("classifier_results/random_forest_model.joblib")
pipeline = bundle["pipeline"]
le       = bundle["label_encoder"]
features = bundle["feature_names"]
```

---

## Tips

- **Rerunning is fast** — light curves are cached in `lc_cache/` after the first download.
- **Changing targets** — edit only the `CONFIG` dict at the top of each script.
- **Parallel downloads** — Pipeline A2 uses 3 threads by default (`"download_threads": 3`). Increase carefully to stay within MAST rate limits.
- **Fainter stars** — increase `"tmag_max"` to 15.0 to probe more candidates per field.
