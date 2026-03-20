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

| Metric | SIND | Best from All Competitor |
|--------|------|-----------------|
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

```
Raw Logs → Behavioral Feature Engineering
        → Mutual Information Feature Selection (X_sel)
        → SMOTE Class Balancing (X_bal)
        → Semi-Naive Bayesian Network (MAP/BDeu)
        → Calibrated Risk Score P(Label=1 | evidence)
        → Binary Prediction ŷ ∈ {0, 1}
```

### Semi-Naive Bayesian Network

The core model combines:
- **Direct label dependence**: `Label → feat_i` for all features (Naive Bayes baseline)
- **Intra-domain dependency chains**: directed edges between features within the same behavioral domain (device, logon, email, file, web)

This captures behavioral correlations (e.g., USB connect → file copy) without the exponential CPT growth of fully connected structures.

### Behavioral Domains & Selected Features (Top-24)

| Domain | Representative Features |
|--------|------------------------|
| `device` | `device_usb_connects`, `device_usb_disconnects`, `device_max_daily_device_events`, `device_offhour_device_ratio`, `device_active_days`, ... |
| `logon` | `logon_offhour_logon_count`, `logon_unique_pcs`, `logon_failed_logons`, ... |
| `file` | `file_total_file_events`, `file_unique_files`, `file_unique_file_types`, ... |
| `email` | `email_email_total_attachments`, `email_total_recipients`, ... |
| `http` / `web` | Web browsing behavioral aggregates |

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
├── SIND _Semi_Naive_InsiderDetector.ipynb   # Main notebook — full pipeline
├── 4.2_meses_18.zip                         # Preprocessed CERT r4.2 splits
│   ├── X_train.csv
│   ├── X_test.csv
│   ├── y_train.csv
│   └── y_test.csv
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.8+
- Jupyter Notebook or Google Colab

### Installation

```bash
pip install pgmpy scikit-learn networkx matplotlib seaborn xgboost imbalanced-learn
```

Or in a Colab/notebook cell:

```python
!pip install pgmpy scikit-learn networkx matplotlib seaborn xgboost imblearn
```

### Running the Notebook

1. Clone the repository:
   ```bash
   git clone https://github.com/CESAR-CISSA/SIND.git
   cd SIND
   ```

2. Extract the dataset:
   ```bash
   unzip 4.2_meses_18.zip
   ```

3. Open and run the notebook:
   ```bash
   jupyter notebook "SIND _Semi_Naive_InsiderDetector.ipynb"
   ```
   
   Update the data paths at the top of the notebook to point to the extracted CSVs:
   ```python
   X_train = pd.read_csv('./4.2_meses_18/X_train.csv')
   X_test  = pd.read_csv('./4.2_meses_18/X_test.csv')
   y_train = pd.read_csv('./4.2_meses_18/y_train.csv').squeeze()
   y_test  = pd.read_csv('./4.2_meses_18/y_test.csv').squeeze()
   ```

---

## Dataset

Experiments use the **CERT Insider Threat Dataset r4.2**, developed by the Software Engineering Institute (SEI) at Carnegie Mellon University. It simulates the daily activities of 1,000 users over 17 months across five event sources: authentication, removable device usage, web browsing, email, and file operations. Ground-truth labels cover malicious scenarios such as data exfiltration by departing employees and system sabotage by disgruntled administrators.

The preprocessed train/test splits (`4.2_meses_18.zip`) are included directly in this repository. The original raw logs can be obtained from the [CERT dataset page](https://resources.sei.cmu.edu/library/asset-view.cfm?assetid=508099).

> **Training distribution (after SMOTE):** 651 normal / 279 anomalous  
> **Test distribution (imbalanced):** 279 normal / 21 malicious

---

## Notebook Structure

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
  booktitle = {EEE International Conference on Systems, Man, and Cybernetics (SMC)},
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
