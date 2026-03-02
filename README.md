# UNSW-NB15 Train/Val/Test Splits - Sequential Temporal

## Overview

This repository contains the UNSW-NB15 network intrusion detection dataset split into training,
validation, and test sets using sequential temporal ordering. The methodology prioritizes
chronological integrity (train on historical data, validate/test on contemporary data),
reflecting realistic IDS deployment scenarios.

**Dataset composition:**

- Total records: 2,054,316 network flows (after deduplication)
- Training set: 1,232,589 records (60%)
- Validation set: 410,863 records (20%)
- Test set: 410,864 records (20%)
- Features: 49 network flow metrics
- Classes: Binary (normal/attack) + 10 attack categories
- Duplicates removed: 477,359 exact duplicate records

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

The original UNSW-NB15 dataset (2,540,047 records) contained 477,359 exact duplicate records
(~18.8% duplication rate). These duplicates were removed before splitting to eliminate potential
data leakage. The deduplicated dataset (2,054,316 records) was used for all subsequent splits.

## Splitting Methodology

### Sequential Temporal Split

The deduplicated dataset is split sequentially by Unix timestamp (stime) to preserve
chronological ordering:

1. **Train (60%)**: Historical data (1421927377 → 1424229505)
2. **Validation (20%)**: Recent history (1424229505 → 1424245853)
3. **Test (20%)**: Current/future data (1424245853 → 1424262068)

This approach reflects realistic IDS deployment where models are trained on historical
traffic and evaluated on contemporary unseen traffic.

### Algorithm

#### Step 1: Sort by timestamp

- Sort all records by Unix timestamp (stime) in ascending order
- Total timeline: 648.5 hours

#### Step 2: Sequential partitioning

- Train: Records 0-60% (chronologically earliest)
- Validation: Records 60-80% (chronologically intermediate)
- Test: Records 80-100% (chronologically latest)

#### Step 3: Maintain separation

- No temporal overlap between splits (strictly sequential)
- Test set represents definitively future/unseen data relative to training

### Results

**Class distribution (varies due to temporal clustering of attacks):**

| Split | Normal | Attack | Normal % | Attack % | Records |
| ------- | -------- | -------- | ---------- | ---------- | --------- |
| Original | 1,954,709 | 99,607 | 95.15% | 4.85% | 2,054,316 |
| Train | 1,202,097 | 30,492 | 97.53% | 2.47% | 1,232,589 |
| Val | 375,118 | 35,745 | 91.30% | 8.70% | 410,863 |
| Test | 377,494 | 33,370 | 91.88% | 8.12% | 410,864 |

**Temporal distribution characteristics:**

- Attacks concentrate in later part of timeline (validation/test periods)
- Training data has lower attack density (2.47%) → requires careful evaluation
- Validation/Test have higher attack density (8-9%) → more representative of attack scenarios
- This clustering is **realistic** - reflects how real attacks may occur sporadically
- Data leakage: **ZERO** (verified - no overlapping records across strictly sequential splits)

## Files

```text
unsw_nb15_train.csv          Training set - historical (1,232,589 records)
unsw_nb15_val.csv            Validation set - recent (410,863 records)
unsw_nb15_test.csv           Test set - current/future (410,864 records)
unsw_nb15.csv                Full cleaned dataset before split (2,054,316 records)
unsw_nb15_split_report.csv   Detailed split statistics
unsw_nb15_column_report.csv  Column metadata and properties
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
- **No duplicate records** (477,359 duplicates removed from original dataset)
- **Zero data leakage** (no overlapping records between train/val/test splits)

### Class Imbalance and Temporal Clustering

The dataset exhibits severe class imbalance overall (95.15% normal, 4.85% attack).
Sequential temporal split reveals **attacks cluster in later time periods**:

- **Training set**: 97.53% normal, 2.47% attack (sparse attack signals)
- **Validation/Test**: ~91% normal, ~8-9% attack (higher attack density)

**Recommendations:**

1. **Training strategy**: Apply ADASYN oversampling (1:1 ratio) to training set only to
   compensate for attack scarcity in historical data
2. **Evaluation**: Use stratified validation/test metrics to account for temporal-dependent
   class distribution
3. **Model tuning**: Use validation set to monitor for overfitting on attack-rich data

See Usage section for oversampling example code.

## Recommended Evaluation Metrics

Due to class imbalance (varying by temporal period), do not rely on accuracy alone:

- F1-Score (weighted and macro)
- Precision-Recall curves
- ROC-AUC score
- Per-class precision and recall
- Matthews Correlation Coefficient (MCC)

## License

This dataset maintains UNSW-NB15 original licensing terms. Academic and research use
requires attribution to Moustafa & Slay (2015).
