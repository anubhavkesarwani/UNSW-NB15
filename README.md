# UNSW-NB15 Dataset — Cleaned & Split for NIDS Research

| Property | Value |
|----------|-------|
| Records (cleaned) | 2,054,316 |
| Train / Test | 1,643,452 (80%) / 410,864 (20%) |
| Features | 49 |
| Classes | Binary (normal/attack) + 10 attack categories |
| Split method | Sequential temporal (chronological) |
| Data leakage | Zero (verified) |

## Attribution

> Moustafa, N., & Slay, J. (2015). UNSW-NB15: A comprehensive data set for network
> intrusion detection systems. *2015 Military Communications and Information Systems
> Conference (MilCIS)*, pp. 1–6. IEEE.

Dataset source: [UNSW Canberra Cyber Security](https://research.unsw.edu.au/projects/unsw-nb15-dataset)

## Preparation Pipeline

Run via `scripts/prepare_unswnb15_enhanced.py`. Single script, no dependencies beyond
pandas and numpy.

### Steps & Justifications

| Step | What | Why |
|------|------|-----|
| Merge shards | Combine 4 CSV shards using feature names from `UNSW-NB15_features.csv` | Raw dataset is distributed across 4 files without headers |
| Label cleaning | Standardize `label` to 0/1, normalize `attack_cat` (e.g. `Backdoors` → `backdoor`) | Raw data contains inconsistent casing and plural forms |
| FTP field cleaning | Coerce `ct_ftp_cmd`, `is_ftp_login` to int, clip to valid range | Known UNSW-NB15 issue — these fields contain non-numeric values in raw data |
| Categorical standardization | Lowercase + trim `proto`, `state`, `service` | Prevents duplicate categories from casing differences |
| Remove zero-duration flows | Drop records where `dur == 0` (8,372 removed) | Zero-duration flows lack temporal characteristics needed for time-based features |
| Remove duplicate columns | Identify via exact value comparison across all numeric columns | Redundant features add noise without information |
| Remove constant columns | Drop numeric columns with single unique value | Zero variance → zero predictive value |
| Deduplication | Drop exact duplicate rows excluding `stime` (477,359 removed) | Duplicates cause data leakage and inflate metrics [1] |
| Sequential temporal split | Sort by `stime`, take first 80% as train, remaining 20% as test | Chronological splitting prevents temporal leakage and reflects realistic deployment [1][2][3] |

### Why Sequential Temporal Split (Not Random)

Random splitting of network traffic data violates temporal causality — the model sees
"future" traffic during training, producing inflated metrics that don't generalize to
deployment [1][2]. This is well-documented:

- Arp et al. [1] identify random splitting as a key methodological flaw in ML-based security
  systems and recommend time-based partitioning
- Corsini et al. [2] demonstrate that random splits introduce temporal leakage in sequential
  NIDS, leading to overly optimistic results
- El Mahdaouy et al. [3] survey 80+ NIDS papers and list chronological splitting as a
  mandatory best practice for evaluation rigor

### Why 80/20 (Not 70/30 or 60/20/20)

With 2M+ records, even 20% yields ~410K test samples — sufficient for statistically robust
evaluation. A separate validation split is omitted because hyperparameter tuning is better
handled via cross-validation on the training set (using temporal folds), keeping the test
set strictly held out.

### Why No Normalization in This Pipeline

Normalization must be fit on training data only and applied to test data [1]. This is a
training-time operation that depends on the model (tree-based models don't need it).
Baking it into the data prep would lock in one strategy and destroy raw feature values.

## Class Distribution

| Split | Normal | Attack | Normal % | Attack % |
|-------|--------|--------|----------|----------|
| Full | 1,954,709 | 99,607 | 95.15% | 4.85% |
| Train | 1,577,215 | 66,237 | 95.97% | 4.03% |
| Test | 377,494 | 33,370 | 91.88% | 8.12% |

Distribution varies across splits — this is expected and realistic with temporal ordering.
Attacks cluster in later time periods, which mirrors real-world deployment where attack
patterns are non-stationary.

## Files

```
unsw_nb15_train.csv          Train set (1,643,452 records)
unsw_nb15_test.csv           Test set (410,864 records)
unsw_nb15.csv                Full cleaned dataset before split
unsw_nb15_split_report.csv   Split statistics
unsw_nb15_column_report.csv  Column metadata
```

## Downstream Responsibilities

These steps belong in the ML training pipeline, not in data preparation:

- **Feature selection**: Drop `srcip`, `dstip` (IP memorization), `stime`, `ltime`
  (absolute timestamps don't generalize)
- **Encoding**: One-hot or target-encode `proto`, `state`, `service`
- **Normalization**: Fit scaler on train, transform test
- **Class imbalance**: ADASYN/SMOTE on train only, or use `class_weight='balanced'`

## References

[1] Arp, D., Quiring, E., Pendlebury, F., Warnecke, A., Pierazzi, F., Wressnegger, C.,
Cavallaro, L., & Rieck, K. (2022). Dos and don'ts of machine learning in computer security.
*31st USENIX Security Symposium*, pp. 3971–3988.

[2] Corsini, A., Yang, S.J., & Apruzzese, G. (2021). On the evaluation of sequential
machine learning for network intrusion detection. *Proceedings of the 16th International
Conference on Availability, Reliability and Security (ARES)*, pp. 1–10.

[3] El Mahdaouy, A., Ait Yahia, I., Oualil, S., & Berrada, I. (2026). Deep learning for
contextualized NetFlow-based network intrusion detection: Methods, data, evaluation and
deployment. *arXiv:2602.05594*.

## License

UNSW-NB15 original licensing terms apply. Academic use requires attribution to
Moustafa & Slay (2015).
