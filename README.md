# UNSW-NB15 Train/Val/Test Splits - Temporal Stratified

## Overview

This repository contains the UNSW-NB15 network intrusion detection dataset split into training,
validation, and test sets using temporal stratified sampling. The methodology preserves both
chronological integrity and statistical class balance, ensuring reliable model evaluation.

**Dataset composition:**

- Total records: 2,540,047 network flows
- Training set: 1,524,027 records (60%)
- Validation set: 508,010 records (20%)
- Test set: 508,010 records (20%)
- Features: 49 network flow metrics
- Classes: Binary (normal/attack) + 10 attack categories

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

## Splitting Methodology

### Temporal Stratification

The dataset was split using temporal stratified sampling to balance two competing requirements:

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
| Train | 1,331,257 | 192,770 | 87.35% | 12.65% | 1,524,027 |
| Val | 443,753 | 64,257 | 87.35% | 12.65% | 508,010 |
| Test | 443,754 | 64,256 | 87.35% | 12.65% | 508,010 |

**Stratification quality:**

- Label distribution difference: 0.0000-0.0001%
- Attack category distribution difference: 0.0002%
- Imbalance ratio (Normal:Attack): 6.91:1 maintained

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

# Handle class imbalance in training
from sklearn.ensemble import RandomForestClassifier
model = RandomForestClassifier(class_weight='balanced')
```

## Data Quality

- Zero missing values
- No infinite values
- All numeric fields within valid ranges
- Categorical values normalized (lowercase, trimmed)
- Consistent data types across splits

## Recommended Evaluation Metrics

Due to 12.65% attack density, do not rely on accuracy alone:

- F1-Score (weighted)
- Precision-Recall curves
- ROC-AUC
- Per-class precision and recall

## License

This dataset maintains UNSW-NB15 original licensing terms. Academic and research use
requires attribution to Moustafa & Slay (2015).
