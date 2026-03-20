# SIND — Semi-Naive Insider Detector

> **SIND: A Semi-Naive Bayesian Approach for Insider Threat Detection**  
> Matheus V. P. dos Santos, José Edson C. A. Júnior, Damião de O. M. Neto, Pedro H. G. Liberal, Anna C. F. Almeida, Igor F. B. do Rêgo, Anderson M. de Morais, Wellison R. M. Santos, Fernando Aires, Milton Lima, J. R. Campos  
> CESAR / CISSA — Center for Security in Advanced Systems, Recife, Brazil  
> *Funded by EMBRAPII — Project CIS-AFCCT-2024-7-26-2*

---

## Overview

Insider threats represent a critical challenge in organizational cybersecurity: malicious actors with legitimate access can cause significant damage while evading conventional detection mechanisms. Existing approaches typically sacrifice interpretability for accuracy, or rely on large volumes of labeled data that are rarely available in practice.

**SIND** is a probabilistic framework that addresses these limitations through a **Semi-Naive Bayesian Network (SNBN)** combining:

- Multi-source **behavioral feature engineering** (logon, device, email, file, web activity)
- **Synthetic class balancing** via SMOTE to handle extreme label imbalance
- **Probabilistic inference** via MAP estimation with BDeu prior for interpretable, auditable risk scores

The causal graph structure enables security analysts to understand *which behavioral factors* drove each classification decision — a requirement that is currently underaddressed in production detection systems.

---

## Key Results (CERT r4.2)

| Metric | SIND | Best from All Competitors |
|--------|------|---------------------------|
| Accuracy | **0.980** | 0.972 |
| Precision | **0.900** | — |
| Recall | **0.970** | 0.884 |
| F1-Score | **0.930** | 0.870 |
| Detection Rate | **0.950** | 0.840 |
| FPR | 0.018 | 0.015 |
| AUC-ROC | **0.994** | — |
| AUPRC | **0.930** | — |

Evaluated on a strictly imbalanced test set (279 normal / 21 malicious) to reflect a realistic needle-in-a-haystack scenario.

---

## Architecture

SIND operates as a four-stage end-to-end pipeline:

<img width="1449" height="304" alt="Captura de tela 2026-03-20 134018" src="https://github.com/user-attachments/assets/70dc400a-8b3c-482c-aa87-bfe7e122838a" />

### Semi-Naive Bayesian Network

The core model combines:
- **Direct label dependence**: `Label → feat_i` for all features (Naive Bayes baseline)
- **Intra-domain dependency chains**: directed edges between features within the same behavioral domain (device, logon, email, file, web)

This captures behavioral correlations (e.g., USB connect → file copy) without the exponential CPT growth of fully connected structures.

---

## Selected Features

The 24 features used by SIND are derived via Mutual Information analysis across five behavioral domains.

| Feature | Description |
|---------|-------------|
| `device_std_daily_device_events` | Standard deviation of daily device events. |
| `device_mean_daily_device_events` | Mean number of daily device events. |
| `device_max_daily_device_events` | Maximum number of device events in a single day. |
| `device_device_events_total` | Total number of device events. |
| `device_offhour_device_ratio` | Ratio of off-hours device events to total events. |
| `device_offhour_device_count` | Absolute count of off-hours device events. |
| `device_active_days` | Number of days in which the device was active. |
| `device_unique_pcs_used` | Number of distinct PCs used by the user. |
| `device_usb_connects` | Count of USB device connections. |
| `logon_total_logons_day` | Total number of daily logons. |
| `logon_total_logoffs_day` | Total number of daily logoffs. |
| `logon_sessions_total_duration` | Total duration of logon sessions. |
| `logon_open_sessions_mean` | Mean number of simultaneously open sessions. |
| `logon_missing_logoff_count` | Count of logons without corresponding logoff. |
| `logon_logon_to_logoff_ratio` | Ratio between logons and logoffs. |
| `logon_distinct_pcs_used` | Number of distinct PCs accessed through logon. |
| `email_n_email` | Total number of sent/received emails. |
| `email_email_total_attachments` | Total number of email attachments. |
| `email_email_unique_bcc` | Number of unique BCC recipients. |
| `email_email_unique_to` | Number of unique recipients in the To field. |
| `email_email_mean_text_len` | Mean length of email text. |
| `file_pdf_files_accessed` | Number of accessed PDF files. |
| `file_exe_files_accessed` | Number of accessed executable files (`.exe`). |
| `http_unique_urls_visited` | Number of unique visited HTTP URLs. |

---

## Repository Contents

```
SIND/
├── Extração de Features.ipynb               # Step 1 — raw CERT r4.2 logs → user-level feature matrix
├── SIND _Semi_Naive_InsiderDetector.ipynb   # Step 2 — feature selection, SMOTE, BN training & evaluation
├── 4.2_meses_18.zip                         # Preprocessed CERT r4.2 splits (output of Step 1)
│   ├── X_train.csv
│   ├── X_test.csv
│   ├── y_train.csv
│   └── y_test.csv
└── README.md
```

The two notebooks form a sequential pipeline: **`Extração de Features`** processes the raw CERT r4.2 CSV logs into a per-user behavioral feature matrix, and **`SIND`** consumes that matrix to train and evaluate the Semi-Naive Bayesian Network.

---

## Getting Started

### Prerequisites

- Python 3.8+
- Jupyter Notebook or Google Colab
- Raw CERT r4.2 logs (required only if re-running feature extraction — see Option B)

### Installation

```bash
pip install pgmpy scikit-learn networkx matplotlib seaborn xgboost imbalanced-learn
```

Or in a Colab/notebook cell:

```python
!pip install pgmpy scikit-learn networkx matplotlib seaborn xgboost imblearn
```

### Option A — Run from preprocessed splits (recommended)

Use the included `4.2_meses_18.zip` to reproduce the SIND results without re-extracting features.

1. Clone the repository:
   ```bash
   git clone https://github.com/CESAR-CISSA/SIND.git
   cd SIND
   ```

2. Extract the dataset:
   ```bash
   unzip 4.2_meses_18.zip
   ```

3. Open and run the main notebook:
   ```bash
   jupyter notebook "SIND _Semi_Naive_InsiderDetector.ipynb"
   ```

   Update the data paths at the top of the notebook:
   ```python
   X_train = pd.read_csv('./4.2_meses_18/X_train.csv')
   X_test  = pd.read_csv('./4.2_meses_18/X_test.csv')
   y_train = pd.read_csv('./4.2_meses_18/y_train.csv').squeeze()
   y_test  = pd.read_csv('./4.2_meses_18/y_test.csv').squeeze()
   ```

### Option B — Full pipeline from raw logs

To reproduce everything from scratch starting from the original CERT r4.2 event logs:

1. Download the raw CERT r4.2 dataset (see [Dataset](#dataset)) and place the CSVs under `/srv/cert/r4.2/`:
   ```
   /srv/cert/r4.2/
   ├── device.csv
   ├── logon.csv
   ├── file.csv
   ├── email.csv
   └── http.csv
   ```

2. Run **`Extração de Features.ipynb`**. The notebook parses each event source, computes per-user behavioral aggregates, joins all domain tables on `user`, and assigns binary labels from the ground-truth metadata.

3. Use the resulting feature matrix as input to **`SIND _Semi_Naive_InsiderDetector.ipynb`**.

---

## Dataset

Experiments use the **CERT Insider Threat Dataset r4.2**, developed by the Software Engineering Institute (SEI) at Carnegie Mellon University. It simulates the daily activities of 1,000 users over 17 months across five event sources: authentication, removable device usage, web browsing, email, and file operations. Ground-truth labels cover malicious scenarios such as data exfiltration by departing employees and system sabotage by disgruntled administrators.

The preprocessed train/test splits (`4.2_meses_18.zip`) are included directly in this repository. The original raw logs can be obtained from the [CERT dataset page](https://resources.sei.cmu.edu/library/asset-view.cfm?assetid=508099).

> **Training distribution (after SMOTE):** 651 normal / 279 anomalous  
> **Test distribution (imbalanced):** 279 normal / 21 malicious

---

## Notebook Structure

### `Extração de Features.ipynb` — Feature Engineering

| Step | Description |
|------|-------------|
| Device features | Parse `device.csv`; compute connect/disconnect counts, off-hours ratios, active days, unique PCs |
| Logon features | Parse `logon.csv`; compute session durations, logon/logoff ratios, missing logoff counts |
| File features | Parse `file.csv`; compute total/unique file accesses, PDF and executable file counts |
| Email features | Parse `email.csv`; compute attachment totals, unique BCC/To recipients, mean text length |
| HTTP features | Parse `http.csv`; compute total requests and unique URLs per user |
| Feature merging | Left-join all domain tables on `user`; fill missing values; apply domain prefix (`device_*`, `logon_*`, …) |
| Labeling | Assign binary labels (`1` = malicious, `0` = normal) from ground-truth metadata |

### `SIND _Semi_Naive_InsiderDetector.ipynb` — Detection Pipeline

| Section | Description |
|---------|-------------|
| 1. Data Loading | Load train/test CSVs, inspect class distribution |
| 2. Feature Selection | Mutual Information analysis per behavioral domain |
| 3. Discretization | `KBinsDiscretizer` (quantile strategy, 5 bins) |
| 4. Latent Features | K-Means clustering per domain for intermediate nodes |
| 5. Network Architectures | Define Naive, Hierarchical, Semi-Naive, TAN, Hybrid edge sets |
| 6. Training & Evaluation | BN training with MLE/MAP; comparison vs sklearn baselines |
| 7. Confusion Matrix | Best model analysis |
| 8. Explainability Metrics | Mutual Information, KL Divergence, permutation importance |
| 9. Network Visualization | Node-size-proportional influence graph |
| 10. Architecture Comparison | Side-by-side visualization of all topologies |
| 11. CPD Analysis | How top nodes influence predictions |
| 12. Cross-Validation | Robustness evaluation |

---

## Reproducibility

All experiments use a fixed random seed (`random_state = 42`). The SMOTE ratio targets 30% minority class relative to the majority.

Key hyperparameters:
- `N_BINS = 5`, `STRATEGY = 'quantile'` (discretization)
- `prior_type = 'BDeu'`, `equivalent_sample_size = 5` (MAP estimation)
- `N_LATENT_STATES = 3` (latent node clustering)

---

## Citation

If you use SIND in your research, please cite:

```bibtex
@inproceedings{santos2025sind,
  title     = {{SIND}: A Semi-Naive {B}ayesian Approach for Insider Threat Detection},
  author    = {Santos, Matheus V. P. dos and J{\'u}nior, Jos{\'e} Edson C. A. and
               Neto, Dami{\~a}o de O. M. and Liberal, Pedro H. G. and
               Almeida, Anna C. F. and R{\^e}go, Igor F. B. do and
               Morais, Anderson M. de and Santos, Wellison R. M. and
               Aires, Fernando and Lima, Milton and Campos, J. R.},
  booktitle = {IEEE International Conference on Systems, Man, and Cybernetics (SMC)},
  year      = {2026},
  note      = {Funded by EMBRAPII, Project CIS-AFCCT-2024-7-26-2}
}
```

---

## Acknowledgements

This work was supported by **EMBRAPII** through the project *Identifying and interpreting behavioral deviations of malicious users or those linked to cybercrime* (Project Code: CIS-AFCCT-2024-7-26-2).

Affiliations:
- **CESAR / CISSA** — Center for Advanced Studies and Systems / Center for Security in Advanced Systems, Recife, Pernambuco, Brazil
- **CISUC / University of Coimbra** — Centre for Informatics and Systems of the University of Coimbra, Portugal

---

## License

This repository is released for research and academic use. Please refer to [LICENSE](LICENSE) for details.
