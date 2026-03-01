# UNSW-NB15 Train/Val/Test Splits - Temporal Stratified

## Overview

This repository contains the UNSW-NB15 network intrusion detection dataset split into training,
validation, and test sets using temporal stratified sampling. The methodology preserves both
chronological integrity and statistical class balance, ensuring reliable model evaluation.

**Dataset composition:**

- Total records: 2,059,408 network flows (after deduplication)
- Training set: 1,235,644 records (60%)
- Validation set: 411,882 records (20%)
- Test set: 411,882 records (20%)
- Features: 49 network flow metrics
- Classes: Binary (normal/attack) + 10 attack categories
- Duplicates removed: 480,639 exact duplicate records

## Dataset Attribution

The UNSW-NB15 dataset is provided by the Cyber Range and Security Operations Centre (CySEC)
at the University of New South Wales (UNSW Sydney).

**Original citation:**

```text
Moustafa, N., & Slay, J. (2015). UNSW-NB15: a comprehensive data set for network
intrusion detection systems (UNSW-NB15 network data set). In 2015 Military
Communications and Information Systems Conference (MilCIS) (pp. 1-6). IEEE.
```

**Official dataset:**
[UNSW-NB15: Network Intrusion Detection Datasets](https://www.unsw.adfa.edu.au/unsw-canberra/academic/school-engineering/school-engineering-research/cybersecurity/UNSW-NB15-Datasets)

## Data Preparation

### Deduplication

The original UNSW-NB15 dataset (2,540,047 records) contained 480,639 exact duplicate records
(~19% duplication rate). These duplicates were removed before splitting to eliminate potential
data leakage. The deduplicated dataset (2,059,408 records) was used for all subsequent splits.

## Splitting Methodology

### Temporal Stratification

The deduplicated dataset was split using temporal stratified sampling to balance two competing
requirements:

1. **Chronological integrity**: Preserve temporal ordering to reflect realistic deployment
   scenarios (train on historical data, test on contemporary data)
2. **Class distribution stability**: Maintain identical class ratios across splits to ensure
   fair model evaluation

### Algorithm

#### Step 1: Timeline division

- Sort records by Unix timestamp (stime)
- Divide 648.5-hour timeline into 25 equal windows

#### Step 2: Stratified sampling within windows

- Stratify by label (0=normal, 1=attack) and attack_cat (10 types)
- Proportionally sample: 60% train, 20% validation, 20% test

#### Step 3: Chronological assembly

- Concatenate stratified samples from all windows
- Preserve temporal scatter across splits

### Results

**Class distribution (identical across splits):**

| Split | Normal | Attack | Normal % | Attack % | Records |
| ------- | -------- | -------- | ---------- | ---------- | --------- |
| Train | 1,175,858 | 59,786 | 95.16% | 4.84% | 1,235,644 |
| Val | 391,953 | 19,929 | 95.16% | 4.84% | 411,882 |
| Test | 391,954 | 19,928 | 95.16% | 4.84% | 411,882 |

**Stratification quality:**

- Label distribution difference: 0.0000-0.0001%
- Attack category distribution difference: 0.0003%
- Imbalance ratio (Normal:Attack): 19.67:1 maintained
- Data leakage: **ZERO** (verified - no overlapping records across splits)

## Files

```text
unsw_nb15_train.csv          Training set (1,524,027 records)
unsw_nb15_val.csv            Validation set (508,010 records)
unsw_nb15_test.csv           Test set (508,010 records)
```

## Features (49 total)

**Network flow:** srcip, dstip, sport, dsport, proto, state, service

**Basic traffic:** dur, sbytes, dbytes, sttl, dttl, sloss, dloss, sload, dload

**Packet characteristics:** spkts, dpkts, swin, dwin, stcpb, dtcpb, smeansz, dmeansz

**Timing & jitter:** stime, ltime, sjit, djit, sintpkt, dintpkt, tcprtt, synack, ackdat,
trans_depth, res_bdy_len

**Aggregation features:** is_sm_ips_ports, ct_state_ttl, ct_flw_http_mthd, is_ftp_login,
ct_ftp_cmd, ct_srv_src, ct_srv_dst, ct_dst_ltm, ct_src_ltm, ct_src_dport_ltm,
ct_dst_sport_ltm, ct_dst_src_ltm

**Target variables:** label (0=normal, 1=attack), attack_cat (9 attack types + normal)

## Usage

```python
import pandas as pd

# Load datasets
train = pd.read_csv('unsw_nb15_train.csv')
val = pd.read_csv('unsw_nb15_val.csv')
test = pd.read_csv('unsw_nb15_test.csv')

# Recommended: Handle severe class imbalance with ADASYN oversampling
# (Apply only to training set for honest validation/test evaluation)
# from imblearn.over_sampling import ADASYN
# X_train, y_train = ADASYN(sampling_strategy=1.0).fit_resample(
#     train.drop('label', axis=1), train['label']
# )
# train_balanced = X_train.copy()
# train_balanced['label'] = y_train

# Alternative: Use class weights in model
from sklearn.ensemble import RandomForestClassifier
model = RandomForestClassifier(class_weight='balanced')
```

## Data Quality

- Zero missing values
- No infinite values
- All numeric fields within valid ranges
- Categorical values normalized (lowercase, trimmed)
- Consistent data types across splits
- **No duplicate records** (480,639 duplicates removed from original dataset)
- **Zero data leakage** (no overlapping records between train/val/test splits)

### Class Imbalance Note

The dataset exhibits severe class imbalance (95.16% normal, 4.84% attack) due to duplicate
removal prioritizing data integrity. **Recommendation: Apply ADASYN oversampling (1:1 ratio)
to training set only** to improve model learning without distorting val/test metrics. See
Usage section for example code.

## Recommended Evaluation Metrics

Due to 4.84% attack density (highly imbalanced), do not rely on accuracy alone:

- F1-Score (weighted and macro)
- Precision-Recall curves
- ROC-AUC score
- Per-class precision and recall
- Matthews Correlation Coefficient (MCC)

## License

This dataset maintains UNSW-NB15 original licensing terms. Academic and research use
requires attribution to Moustafa & Slay (2015).
