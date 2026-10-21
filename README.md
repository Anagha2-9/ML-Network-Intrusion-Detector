# ML Network Intrusion Detector

A beginner-friendly machine learning project that classifies network traffic as
**Benign** or **Malicious** using Scikit-learn on the NSL-KDD benchmark dataset.

---

## Problem Statement

Networks are constantly probed and attacked. A **Network Intrusion Detection System (IDS)**
monitors traffic and raises an alarm when suspicious activity is detected. This project
trains two supervised ML models — Logistic Regression and Random Forest — on labelled
network-flow records and evaluates their ability to distinguish normal traffic from attacks.
The positive class is **Malicious = 1**; the key metric is **Recall** (missed attacks are
more dangerous than false alarms).

---

## Dataset — NSL-KDD

| Property | Detail |
|----------|--------|
| Name | NSL-KDD |
| Official page | https://www.unb.ca/cic/datasets/nsl.html |
| GitHub mirror | https://github.com/defcom17/NSL_KDD |
| Training file | `KDDTrain+.txt` |
| Test file | `KDDTest+.txt` |
| Features | 41 features per connection record (40 used after dropping the constant column num_outbound_cmds) |
| Target | Binary: 0 = Benign (`normal`), 1 = Malicious (all attack types) |

`KDDTest+` intentionally contains attack subtypes not present in `KDDTrain+`, which
is why test scores are lower than validation scores — this is expected behaviour.

---

## Technologies

| Library | Purpose |
|---------|---------|
| Python 3.9+ | Core language |
| Jupyter Notebook / Google Colab | Interactive environment |
| pandas | Data loading and manipulation |
| NumPy | Numerical operations |
| scikit-learn | Pipelines, preprocessing, models, metrics |
| Matplotlib | Charts and figures |
| Seaborn | Heatmaps and styled plots |
| joblib | Model serialisation |

---

## Folder Structure

```
NetworkIntrusionDetection/
├── Anagha_NetworkIntrusionDetection.ipynb   ← main notebook
├── requirements.txt                          ← Python dependencies
├── README.md                                 ← this file
├── models/
│   ├── lr_pipeline.pkl                       ← saved Logistic Regression pipeline
│   ├── rf_pipeline.pkl                       ← saved Random Forest pipeline
│   └── best_pipeline.pkl                     ← best model (chosen by validation F1)
├── KDDTrain+.txt                             ← downloaded on first run
├── KDDTest+.txt                              ← downloaded on first run
├── class_distribution.png                    ← EDA figure
├── attack_types.png                          ← EDA figure
├── confusion_matrices.png                    ← evaluation figure
├── model_comparison.png                      ← evaluation figure
├── feature_importances.png                   ← evaluation figure
└── roc_curve.png                             ← evaluation figure
```

---

## Setup & Run

### Local machine

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Launch Jupyter
jupyter notebook

# 3. Open Anagha_NetworkIntrusionDetection.ipynb
# 4. Run → Restart and Run All
```

The notebook downloads `KDDTrain+.txt` and `KDDTest+.txt` automatically on first run.
If the download fails, get the files from https://github.com/defcom17/NSL_KDD and place
them in the same folder as the notebook.

### Google Colab

1. Upload `Anagha_NetworkIntrusionDetection.ipynb` to Colab (File → Upload notebook).
2. Run the first cell — it installs all dependencies automatically.
3. If the dataset download fails, upload `KDDTrain+.txt` and `KDDTest+.txt` via the
   Files panel (folder icon in the left sidebar), then re-run the download cell.
4. Runtime → Run all.

---

## Results

All numbers are taken directly from the notebook's saved cell outputs.

### Validation split (20% of KDDTrain+) — used for model selection

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|-------|----------|-----------|--------|----|---------|
| Logistic Regression | 0.9746 | 0.9751 | 0.9701 | 0.9726 | 0.9967 |
| Random Forest | 0.9990 | 0.9996 | 0.9982 | 0.9989 | 1.0000 |

### Official test set (KDDTest+) — final honest evaluation

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|-------|----------|-----------|--------|----|---------|
| Logistic Regression | 0.7547 | 0.9170 | 0.6258 | 0.7439 | 0.7910 |
| Random Forest | 0.7714 | 0.9677 | 0.6190 | 0.7551 | 0.9648 |

> **Why the gap?** `KDDTest+` contains novel attack types not seen during training.
> The drop is expected and reflects real-world generalisation difficulty.
> Random Forest is selected as the best model (higher validation F1: 0.9989 vs 0.9726).

---

## Key Design Decisions

- **No data leakage** — `StandardScaler` statistics are fitted only on training data
  inside a `Pipeline`; the same fitted transformer is applied to the test set.
- **ColumnTransformer** — `OneHotEncoder(handle_unknown='ignore')` for the three
  categorical features (`protocol_type`, `service`, `flag`); `StandardScaler` for all
  37 numeric features.
- **`difficulty_level` dropped** — dataset metadata, not a real network feature.
- **`num_outbound_cmds` dropped** — constant column (all zeros), carries no information.
- **`class_weight='balanced'`** — both models adjust for class imbalance automatically.
- **Fixed random seed 42** — results are fully reproducible.

## Limitations

- On KDDTest+, recall is about 0.62, so roughly 38% of attacks are missed. Unseen attack types are the main cause.
- NSL-KDD is an older dataset derived from KDD'99 and does not reflect modern traffic.
- The default 0.5 decision threshold was used; lowering it would raise recall at the cost of more false alarms.
- The binary label hides the attack type (DoS, Probe, R2L, U2R).
