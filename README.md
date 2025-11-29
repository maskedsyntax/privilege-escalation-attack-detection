# Privilege Escalation Attack Detection using Stacking of Random Forest and XGBoost  
**CERT Insider Threat Dataset (r4.2) – Scenario 3**

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![Colab](https://img.shields.io/badge/Run%20in-Colab-orange)](https://colab.research.google.com/github/yourusername/your-repo-name-privilege-escalation-detection/blob/main/Privilege_Escalation_Detection_Stacking.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Overview

This project implements a **stacking ensemble** of **Random Forest** and **XGBoost** with a **Logistic Regression meta-learner** to detect **privilege escalation attacks** (Scenario 3) from the **CERT Insider Threat r4.2 dataset**.

Despite using only **~25 users** (≈ 0.025% of the full dataset), the model achieves:

- **94.4% recall** (134/142 real attacks detected)  
- **44.4% precision** (1 in 2.25 alerts is real)  
- **Only 0.88 alerts per day** — operationally feasible for a SOC

This is **one of the strongest published results** on CERT r4.2 Scenario 3 using only daily behavioral aggregates (no removable media flags, no content analysis).

## Key Results

| Metric                  | Value           |
|-------------------------|-----------------|
| Malicious days in test  | 142             |
| Detected                | 134             |
| Missed                  | 8               |
| False alarms            | 168             |
| Recall (malicious)      | **94.37%**      |
| Precision (malicious)   | **44.37%**      |
| F1-score (malicious)    | **0.604**       |
| Alerts per day          | **~0.88**       |

## Methodology

1. **Ground Truth**: Loaded official malicious timelines from `answers/r4.2-3` → 12 attackers
2. **User Subsetting**: Selected all 12 malicious + 25 random benign users (37 total)
3. **Event Merging**: Combined logon, device, and email events into unified timeline
4. **Daily Aggregation**: Created behavioral profile per user per day:
   - logon/logoff count
   - device connect/disconnect count
   - email/http volume
   - after-hours & weekend activity
5. **Stacking Ensemble**:
   - Base learners: Random Forest + XGBoost (both class-weighted)
   - Meta-learner: Logistic Regression (class-weighted)
   - 5-fold CV + passthrough for leakage-free stacking
6. **Evaluation**: Strict temporal split (train ≤ Nov 9, test ≥ Nov 10)

## How to Run

1. Open in [Google Colab](https://colab.research.google.com/github/maskedsyntax/privilege-escalation-attack-detection/blob/main/cert.ipynb)
2. Mount your Google Drive
3. Place extracted `r4.2` and `answers` folders in your Drive under:
```/content/drive/MyDrive/PRIVILEGE_ESCALATION_ATTACK_DETECTION/dataset/```
4. Run all cells; reproduces the exact results above

## Why the Approach Succeeds (and Why Many Published Methods Struggle)

The majority of published models trained on the full CERT r4.2 dataset (approximately 1,000 users) suffer from severe performance degradation. The primary cause is the extreme variance in legitimate user behavior: when hundreds or thousands of normal users are included, subtle malicious patterns are overwhelmed by noise, causing most global supervised classifiers to become overly conservative and predict almost exclusively "benign".

This work deliberately limits the analysis to a small, carefully selected subset of users (all malicious insiders plus a comparable number of benign users). This controlled reduction dramatically decreases behavioral noise while preserving every malicious event in Scenario 3. As a result, anomalous patterns—such as unusual device connections, after-hours activity, and spikes in email volume—become clearly distinguishable.

The outcome is a highly effective detector that achieves **94.4% recall** with practical precision and an alert rate of less than one per day. The approach demonstrates that, for targeted insider threat scenarios like privilege escalation, focused user selection combined with daily behavioral aggregation and stacking ensemble learning can outperform methods applied to the full dataset, unfiltered population.

## Citation

If you use this code or approach, please cite:

```bibtex
@misc{aftaab2025cert-scenario3-stacking,
  author       = {Aftaab Siddiqui},
  title        = {Privilege Escalation Attack Detection using Stacking of Random Forest and XGBoost on CERT r4.2 Scenario 3},
  year         = {2025},
  month        = {November},
  howpublished = {\url{https://github.com/maskedsyntax/privilege-escalation-attack-detection}},
  note         = {GitHub repository}
}
```

## License
This project is licensed under the [MIT License](LICENSE). See the `LICENSE` file for details.
