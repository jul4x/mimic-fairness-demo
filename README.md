# mimic-fairness-demo

A methodology prototype for evaluating demographic disparities in clinical prediction models, built on the MIMIC-IV Clinical Database Demo.

## What this is

A learning project. The goal is to build, end to end, a small but honest pipeline that:

1. Trains a baseline model (logistic regression) to predict in-hospital mortality from the first 24 hours of an ICU stay.
2. Audits the model's performance across demographic subgroups to surface potential disparities.

It uses the publicly available **MIMIC-IV Demo** (~100 patients) — small enough to iterate on locally, large enough to set up a real pipeline. The subgroup numbers will be too small for reliable fairness statistics; the value of the project is the **method**, not the numbers. A scaled-up version on the full MIMIC-IV is planned, pending PhysioNet credentialing.

## Why this question

In-hospital mortality from the first 24h is one of the most studied tasks on MIMIC, which makes it a good sandbox: I can stand on existing work and focus on the part I actually care about — how the same model behaves across patient groups.

I'm doing this because I'm interested in how clinical AI gets evaluated, not just how it gets built. A model that performs well on average can still fail unevenly across populations, and most published metrics hide that. This is a small attempt to look under the hood.

## Results

The pipeline runs end-to-end on the MIMICC-IV Clinical Data Demo. After applying the inclusion criteria (adult, first ICU stay, LOS ≥ 24h, no missing key fields), the analytic cohort has on the order of 50 stays.

A logistic-regression baseline trained on first-24h vital signs and laboratory summaries (~24 features) produces cross-validated predictions for the full cohort. Headline metrics on out-of-fold predictions — AUC-ROC, AUC-PR, calibration — are reported in `notebooks/05_baseline_model.ipynb`. With ~50 stays and ~24 features the model is overparameterized; predictive performance numbers are illustrative, not estimates.

The fairness audit (`notebooks/06_fairness_audit.ipynb`) stratifies model performance across sex, age band, insurance type, and a collapsed race category. For each subgroup it reports prevalence, predicted positive rate, sensitivity, false positive rate, false negative rate, and AUC where computable, alongside the standard disparity differences (demographic parity, equalized odds) using `fairlearn`. As with the headline metrics, per-subgroup numbers are too small to interpret as population estimates — the deliverable is the audit workflow itself.

## Scope and limitations

- **Demo data** (~100 patients). Subgroup analyses will be statistically noisy. I report them anyway, as a methodology demo.
- **Logistic regression baseline only**. Not the most powerful approach — but simple models well-evaluated are more informative than complex ones poorly understood.
- **No causal claims**. This is a predictive audit, not an analysis of *why* disparities exist.
- **Single dataset**. External validity is not addressed.

## Next steps

Natural extensions, once PhysioNet credentialing comes through and the same code runs on full MIMIC-IV:

- Statistically meaningful subgroup metrics with confidence intervals.
- Better feature extraction (last-value-carried-forward, explicit missingness indicators, broader variable set, severity scores).
- Stronger baselines (XGBoost, simple neural nets) under the same audit framework.
- Intersectional subgroups (e.g., race × insurance) once sample sizes permit.
- A treatment of *what to do* once a disparity is surfaced — bias mitigation methods, the question of when not to deploy a model at all. This is where it gets methodologically interesting and where I'd want to learn from a research group working on this seriously.

## Project structure

```
.
├── README.md
├── requirements.txt
└── notebooks/
    ├── 01_eda_schema.ipynb
    ├── 02_eda_demographics.ipynb
    ├── 03_cohort.ipynb
    ├── 04_features.ipynb
    ├── 05_baseline_model.ipynb
    └── 06_fairness_audit.ipynb
```

## How to reproduce

### 1. Environment

```bash
git clone https://github.com/jul4x/mimic-fairness-demo.git
cd mimic-fairness-demo
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook
```

### 2. Data

The project uses the **MIMIC-IV Clinical Database Demo** (~100 ICU patients), openly available on PhysioNet — no credentialing required.

1. Download the latest version from https://physionet.org/content/mimic-iv-demo/
2. Unzip the archive into the local `data/` directory at the project root.

Expected layout:

```
data/
├── hosp/
│   ├── admissions.csv.gz
│   ├── patients.csv.gz
│   └── ...
└── icu/
    ├── icustays.csv.gz
    ├── chartevents.csv.gz
    └── ...
```

The `data/` directory is gitignored — keeping data local is standard practice (and the same setup will work untouched on the full MIMIC-IV later).

### 3. Notebooks

The notebooks are numbered and meant to be run in order.

## Author

Julie Chasseriaud — 1st-year engineering student at ENSC Bordeaux INP (Cognitique).