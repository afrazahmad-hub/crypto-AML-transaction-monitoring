# AML Crypto Compliance

An end-to-end machine-learning study for prioritizing potentially illicit Bitcoin transactions using the Elliptic transaction graph dataset. The project covers data validation, temporal exploratory analysis, leakage-safe modeling, probability calibration, alert-threshold selection, final evaluation, explainability, and an analyst review queue.

The model output is a review-priority signal. It is not a legal conclusion, a sanctions decision, or an automated basis for filing a report with TRACFIN.

## Project status

The complete workflow is implemented in [`notebooks/main.ipynb`](notebooks/main.ipynb), and the current reproducible run includes generated evaluation and compliance artifacts under [`outputs/`](outputs/).

For a detailed technical walkthrough and interview questions, see [`docs/PROJECT_INTERVIEW_GUIDE.md`](docs/PROJECT_INTERVIEW_GUIDE.md).

The frozen alert policy selected during validation is:

| Component | Selection |
|---|---|
| Model | Class-weighted random forest |
| Feature set | All 166 transaction features |
| Configuration | `rf_deep_leaf_1` |
| Calibration | Sigmoid |
| Alert threshold | `0.09870` |

Final test performance on time steps 35-49:

| Metric | Result |
|---|---:|
| Transactions | 16,670 |
| Illicit prevalence | 6.50% |
| Precision | 56.59% |
| Recall | 73.78% |
| F0.5 | 59.35% |
| Average precision | 77.81% |
| PR-AUC | 77.94% |
| ROC-AUC | 90.27% |
| Alert rate | 8.47% |
| Workload reduction | 91.53% |
| Review-efficiency lift | 8.71x |

These are historical experimental results, not production-performance guarantees. In particular, performance drops sharply from time step 43 onward, demonstrating temporal drift and the need for monitoring, retraining, and human oversight. See [`outputs/step43_period_results.csv`](outputs/step43_period_results.csv) for that comparison and [`outputs/final_test_confidence_intervals.csv`](outputs/final_test_confidence_intervals.csv) for uncertainty estimates.

## Dataset

The project uses the [Elliptic Data Set](https://www.kaggle.com/datasets/ellipticco/elliptic-data-set):

- 203,769 Bitcoin transaction nodes
- 234,355 directed payment-flow edges
- 166 anonymized node features
- 49 chronological time steps
- 42,019 licit, 4,545 illicit, and 157,205 unknown labels

The 166 features comprise 94 local transaction features and 72 aggregated neighborhood features. Unknown labels are excluded from supervised model training and evaluation, then scored separately for the analyst-queue demonstration.

The dataset is not committed to this repository. Follow [`docs/dataset.md`](docs/dataset.md) for download, placement, and credential-safety instructions. The notebook expects these files:

```text
data/raw/elliptic_bitcoin_dataset/
|-- elliptic_txs_features.csv
|-- elliptic_txs_classes.csv
`-- elliptic_txs_edgelist.csv
```

Review and comply with the dataset license and terms on Kaggle.

## Methodology

The notebook implements the project in ordered stages:

1. Configure deterministic seeds, paths, and package information.
2. Load and validate features, labels, transaction IDs, time steps, and graph edges.
3. Examine class imbalance, feature distributions, temporal behavior, graph structure, and the change around time step 43.
4. Exclude unknown labels from supervised learning and create leakage-safe chronological partitions.
5. Establish majority/prevalence baselines and tune class-weighted logistic regression.
6. Tune class-weighted random-forest and histogram-gradient-boosting candidates.
7. Calibrate probabilities and select an alert threshold using validation data only.
8. Apply the frozen policy once to the untouched final test period, with confidence intervals and temporal diagnostics.
9. Produce global importance, example explanations, transaction neighborhoods, unknown-label scores, and a sample analyst queue.
10. Document the human-review workflow, compliance boundaries, limitations, and reproducibility evidence.

Chronological partitions are fixed as follows:

| Partition | Time steps | Purpose |
|---|---|---|
| Training | 1-25 | Model fitting and time-aware tuning |
| Calibration | 26-30 | Probability calibration |
| Threshold validation | 31-34 | Model, calibrator, and alert-threshold selection |
| Final test | 35-49 | One-time frozen-policy evaluation |

This separation prevents later transactions from influencing earlier model-selection decisions and keeps the final test set outside tuning.

## Repository structure

```text
.
|-- notebooks/
|   `-- main.ipynb                  # Complete analysis and modeling workflow
|-- docs/
|   |-- dataset.md                  # Dataset acquisition and placement
|   `-- PROJECT_INTERVIEW_GUIDE.md  # Technical walkthrough and interview prep
|-- outputs/                        # Reproducible tables and handoff artifacts
|-- artifacts/
|   |-- figures/                    # Local generated figures (Git-ignored)
|   `-- models/                     # Local serialized models (Git-ignored)
|-- data/                           # Raw and processed data (Git-ignored)
|-- requirements.txt                # Supported dependency ranges
`-- requirements-lock.txt           # Exact packages from the recorded run
```

## Setup

Python 3.13 is recommended; the recorded run used Python 3.13.5 on Windows.

Create and activate a virtual environment:

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

On macOS or Linux, activate it with:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

For the closest reproduction of the recorded environment, install the lock file:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements-lock.txt
```

The lock file records a Windows environment. On another operating system, or when intentionally resolving newer compatible packages, use:

```bash
python -m pip install -r requirements.txt
```

Optionally register a dedicated Jupyter kernel:

```bash
python -m ipykernel install --user --name aml-crypto-compliance --display-name "Python (AML Crypto Compliance)"
```

## Run the study

1. Download and place the dataset as described in [`docs/dataset.md`](docs/dataset.md).
2. Start Jupyter Lab with `jupyter lab`.
3. Open [`notebooks/main.ipynb`](notebooks/main.ipynb).
4. Select the project environment or registered kernel.
5. Use **Kernel > Restart Kernel and Run All Cells**.
6. Confirm that the final Step 10 checklist reports every condition as `True`.

Step 8 deliberately preserves a single final-test prediction table. Rerunning that cell in the same kernel reuses the saved predictions and verifies that the frozen policy has not changed.

## Generated outputs

| File | Contents |
|---|---|
| [`final_test_summary.csv`](outputs/final_test_summary.csv) | Frozen policy, confusion counts, ranking metrics, alert metrics, and review efficiency |
| [`final_test_confidence_intervals.csv`](outputs/final_test_confidence_intervals.csv) | Bootstrap 95% confidence intervals |
| [`final_test_by_time_step.csv`](outputs/final_test_by_time_step.csv) | Test metrics for each chronological step |
| [`step43_period_results.csv`](outputs/step43_period_results.csv) | Before/after time-step-43 drift comparison |
| [`candidate_operating_points.csv`](outputs/candidate_operating_points.csv) | Step 7 candidate calibrators and thresholds |
| [`calibration_quality.csv`](outputs/calibration_quality.csv) | Calibration diagnostics |
| [`global_feature_importance.csv`](outputs/global_feature_importance.csv) | Permutation-based global feature importance |
| [`analyst_queue_sample.csv`](outputs/analyst_queue_sample.csv) | Ranked unknown-label review sample with risk bands and feature contributions |
| [`compliance_handoff_summary.csv`](outputs/compliance_handoff_summary.csv) | Frozen policy, evaluation metrics, and decision boundaries |
| [`reproducibility_manifest.json`](outputs/reproducibility_manifest.json) | Seeds, versions, data/artifact hashes, partitions, policy, and final metrics |

Input and output SHA-256 hashes in the reproducibility manifest can be used to check that two runs used the same dataset and produced the same artifacts.

## Compliance and model-risk boundaries

- Every alert requires review by a qualified analyst using information beyond the anonymized Elliptic features.
- A high score does not establish money laundering or criminal conduct; a low score does not establish legitimacy.
- The system must not autonomously freeze assets, reject a customer, close an account, or file a TRACFIN report.
- False positives, false negatives, temporal drift, incomplete labels, and selection bias must be monitored.
- Explanations describe model behavior and are not causal evidence.
- Any real deployment requires current legal review, governance, access controls, audit logging, validation on institution-specific data, and an approved escalation process.

The notebook's legal and regulatory discussion is a project snapshot dated 9 September 2026. It is documentation for the experiment, not legal advice.

## Reproducibility notes

- Random seed: `42`
- Tuning and evaluation use chronological rather than random partitions.
- Both average precision and trapezoidal PR-AUC are reported because they are related but not identical.
- The final test set is not used to retune the model, calibrator, or threshold.
- Raw data, credentials, locally serialized models, and generated figures are excluded from Git.
