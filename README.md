# UNSW-NB15 Dataset — Cleaned for NIDS Research

| Property | Value |
|----------|-------|
| Records (cleaned) | 2,054,316 |
| Features | 49 |
| Classes | Binary (normal/attack) + 10 attack categories |

## Attribution

> Moustafa, N., & Slay, J. (2015). UNSW-NB15: A comprehensive data set for network
> intrusion detection systems. *2015 Military Communications and Information Systems
> Conference (MilCIS)*, pp. 1–6. IEEE.

Dataset source: [UNSW Canberra Cyber Security](https://research.unsw.edu.au/projects/unsw-nb15-dataset)

## Preparation Pipeline

Run via single Python script, no dependencies beyond
pandas and numpy.

### Steps & Justifications

| Step | What | Why |
|------|------|-----|
| Load features & merge shards | Combine 4 CSV shards using feature names from `UNSW-NB15_features.csv` | Raw dataset is distributed across 4 files without headers |
| Detect dirty values | Baseline missing values, infinities, and data patterns before cleaning | Establishes starting quality metrics for comparison |
| Apply UNSW-specific cleaning | Standardize `label` to 0/1, normalize `attack_cat`, clean FTP fields, standardize categoricals | Addresses known UNSW-NB15 inconsistencies in labels, attack categories, and FTP data |
| Enforce data types | Convert numeric columns, standardize categoricals, handle infinities/NaNs | Ensures proper data types for analysis and replaces problematic values (e.g., infinities to 0) |
| Comprehensive quality checks | Validate nulls, infinities, timestamps, ports, labels | Confirms data integrity and catches any remaining issues |
| Remove zero-duration flows | Drop records where `dur == 0` (8,372 removed) | Zero-duration flows lack temporal characteristics needed for time-based features |
| Remove duplicate columns | Identify via exact value comparison across numeric columns | Redundant features add noise without information |
| Remove constant columns | Drop numeric columns with single unique value | Zero variance → zero predictive value |
| Before/after comparison | Statistical summary of cleaning impact | Quantifies improvements in data quality |
| Deduplication | Drop exact duplicate rows (477,359 removed) | Duplicates cause data leakage and inflate metrics [1] |
| Save cleaned dataset | Output single `unsw_nb15.csv` with sorted columns and sanity checks | Produces deterministic, validated final dataset |

### Why No Normalization in This Pipeline

Normalization must be fit on training data only and applied to test data [1]. This is a
training-time operation that depends on the model (tree-based models don't need it).
Baking it into the data prep would lock in one strategy and destroy raw feature values.

## Class Distribution

| Normal | Attack | Normal % | Attack % |
|--------|--------|----------|----------|
| 1,954,709 | 99,607 | 95.15% | 4.85% |

## Data Quality Improvements

The cleaning pipeline significantly improves data quality by removing problematic records and standardizing values. Key metrics before and after processing:

| Metric | Before | After | Change |
|--------|--------|-------|--------|
| Records | 2,540,047 | 2,054,316 | -485,731 (-19.11%) |
| Columns | 49 | 49 | 0 |
| Missing Values | 4,996,788 (4.01%) | 0 (0.00%) | -4,996,788 |
| Infinite Values | N/A | 0 | Eliminated |

**Breakdown of Record Reduction:**
- Zero-duration flows removed: 8,372
- Exact duplicate rows removed: 477,359
- Total removed: 485,731

**Data Integrity Validations:**
- All NaN and infinite values eliminated
- Ports clamped to valid ranges (0-65,535)
- Timestamps validated (all positive Unix timestamps)
- Labels standardized to binary (0/1)
- Categorical fields normalized and deduplicated

**Class Distribution Before/After:**

| Class | Before | After | Change | Before % | After % |
|-------|--------|-------|--------|----------|----------|
| Normal (0) | 2,218,764 | 1,954,709 | -264,055 | 87.33% | 95.15% |
| Attack (1) | 321,283 | 99,607 | -221,676 | 12.67% | 4.85% |

**Categorical Feature Diversity:**

After cleaning, categorical features show the following diversity:

- **Protocol (proto)**: 135 unique values (dominated by TCP/UDP, with many rare protocols)
- **Service**: 13 unique values (mostly DNS/HTTP, with unknowns common)
- **State**: 16 unique values (connection states like FIN/CON/INT)

This high cardinality in `proto` suggests careful encoding strategies (e.g., grouping rare protocols) may be needed for ML models.

## Files

```
unsw_nb15.csv                Full cleaned dataset (2,054,316 records, 49 features)
```

## Downstream Responsibilities

These steps belong in the ML training pipeline, not in data preparation. Note that cleaning increases class imbalance (attacks reduced from 12.67% to 4.85%), so robust handling is critical:

- **Feature selection**: Drop `srcip`, `dstip` (IP memorization), `stime`, `ltime`
  (absolute timestamps don't generalize)
- **Encoding**: One-hot or target-encode `proto`, `state`, `service`
- **Normalization**: Fit scaler on train, transform test
- **Class imbalance**: Use `class_weight='balanced'`, ADASYN/SMOTE on train only, or focal loss to address the 95:5 normal:attack ratio

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
