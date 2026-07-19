# CoxPH_screening

Benchmarking and population-screening-impact analysis for multi-modal, Cox proportional-hazards (Cox-PH) cardiovascular disease (CVD) risk prediction models in UK Biobank, built on top of two published risk-prediction frameworks and extended to a two-step "targeted screening" scenario.

> Part of an MSc thesis project. This repository currently contains the core analysis notebook used to reproduce and extend the benchmarking/screening-impact figures described below.

## Overview

Population-scale disease screening needs to balance discriminative accuracy against the practical cost/complexity of data collection. This project benchmarks a set of CVD risk models of varying complexity — from the fully clinical "Qrisk"/"SCORE2"/"PREVENT" risk scores to lower-cost, questionnaire/past-medical-history-only models ("No Needle Model") and models augmented with metabolomics and polygenic risk scores ("Single Blood Draw Model") — and quantifies what each model would mean for a population screening programme in terms of:

- number of individuals classified as high risk,
- number of true CVD events captured in the high-risk group,
- number of CVD events plausibly prevented (assuming a fixed treatment efficacy),
- number needed to screen (NNS) and number needed to treat (NNT) to prevent one event.

It also implements and evaluates a **two-step "target screening"** strategy, in which a low-cost model (the "No Needle Model") is used as first-line triage, and only individuals it flags as medium risk are re-evaluated with a second, more complex model — analogous to a remote pre-check stratification step ahead of an in-person assessment.

## Background & related work

This analysis draws on and combines ideas from two published studies:

1. **Oexner et al.** *"Multi-disease risk prediction models for population-scale personalised screening"* (medRxiv preprint, 2026). This paper defines the multi-disease Cox-PH risk models used here as inputs — the "Clinical Model", "No Needle Model" (past medical history + self-answered questionnaires), "Single Blood Draw Model" (No Needle + ¹H-NMR metabolomics + polygenic risk scores), and the NHS Health Check 1-step/2-step screening scenarios — trained and validated in UK Biobank (and externally in the Estonian Biobank).
2. **Ritchie et al.** *"Combined clinical, metabolomic, and polygenic scores for cardiovascular risk prediction"* (*European Heart Journal*, 2025). This paper's Figure 5 population-screening-impact framework (ΔN­_high-risk, ΔCVD­_high-risk, ΔCVD­_prevented, ΔNNS, ΔNNT, and the comparison between "targeted re-screening" and "prioritising the same number of additional people by the baseline score") is the template for the plots and metrics reproduced and extended in this repository.

**What this repository adds:** the Ritchie et al. screening-impact framework was originally applied only to SCORE2 ± individual biomarkers/PRS. Here, the same framework is applied across the full set of multi-disease/multi-modality models from Oexner et al., benchmarked against QRISK as baseline, and extended with a generalised two-step targeted-screening pipeline (any model can serve as the second-line refinement step for individuals flagged medium-risk by the No Needle Model), including a fair "same number of high-risk individuals" comparison as recommended by Ritchie et al.

## What the notebook does

`Reproduce_ESC_journal_all_models.ipynb` runs the following pipeline for the 10-year CVD endpoint:

1. **Data loading** — downloads pre-computed 10-year survival probability outputs (test-split predictions) for 11 model variants from the UK Biobank Research Analysis Platform (RAP) via `dx download`, along with age-at-recruitment and CVD incident-event files.
2. **Risk stratification** — converts survival probability to 10-year risk percentage and assigns each individual to a low/medium/high risk band, using age-stratified thresholds (different cut-offs for participants aged <50 vs ≥50).
3. **Benchmarking against QRISK** — for each model, computes the number of individuals classified high-risk, the number of true CVD cases within the high-risk group, the estimated number of CVD events prevented (assuming a fixed 20% relative risk reduction from treatment), NNS, and NNT — both in absolute terms and as a delta relative to QRISK — and visualises these as dual-axis (absolute/delta) scatter plots across all models.
4. **Two-step target screening** — uses the "No Needle Model" as a first-line triage step; individuals it places in the medium-risk band are reassigned using each of the other models in turn, and the same set of metrics is recomputed for this two-step scenario.
5. **Matched high-risk-count sensitivity comparison** — as a fairness check (following Ritchie et al.), compares the two-step target-screening scenario against simply selecting the same number of highest-risk individuals ranked by the No Needle Model alone.
6. **Combined visualisation** — merges the single-step "Threshold" scenario and the "NoNeedle_target" (matched-count) scenario into one figure per metric, with QRISK as the fixed baseline.

## Repository structure

```
CoxPH_screening/
└── Reproduce_ESC_journal_all_models.ipynb   # main analysis notebook (data loading → stratification → benchmarking → target screening)
```

> This is currently the primary file in the repository. If additional scripts (e.g. upstream Cox-PH model training, data preprocessing) are added, list and describe them here.

## Data

The notebook expects the following pre-computed CSV files (produced upstream by Cox-PH model fitting on UK Biobank data, not included in this repository due to UK Biobank data-sharing restrictions):

| File | Contents |
|---|---|
| `EN_CVD_clinicalrisk_survival_probs_combined_10y.csv` | Clinical Model 10-year survival probabilities |
| `EN_CVD_pmh_ts_survival_probs_combined_10y.csv` | No Needle Model (PMH + questionnaires) |
| `EN_CVD_qrisk_survival_probs_combined_10y.csv` | QRISK3 |
| `EN_CVD_clinicalrisk_pmh_ts_survival_probs_combined_10y.csv` | Clinical Model + PMH/Questionnaires |
| `EN_CVD_prs_metabolomics_pmh_ts_survival_probs_combined_10y.csv` | Single Blood Draw Model (No Needle + PRS + metabolomics) |
| `EN_CVD_clinicalrisk_prs_metabolomics_survival_probs_combined_10y.csv` | Clinical Model + PRS + metabolomics |
| `EN_CVD_everything_survival_probs_combined_10y.csv` | Full combined model (Clinical + PMH/Questionnaires + Omics) |
| `EN_CVD_nhc_pmh_ts_survival_probs_combined_10y.csv` | NHS Health Check + PMH + Questionnaires |
| `EN_CVD_nhc_prs_metabolomics_pmh_ts_survival_probs_combined_10y.csv` | NHS Health Check + Omics |
| `EN_CVD_prevent_survival_probs_combined_10y.csv` | PREVENT |
| `EN_CVD_score_survival_probs_combined_10y.csv` | SCORE2 |
| `ageAtRecruitment.csv` | Age at UK Biobank recruitment per participant |
| `to_eventCVD.csv` | CVD follow-up time and event status |

Access to the underlying UK Biobank data requires separate, approved access via the [UK Biobank Access Management System](https://www.ukbiobank.ac.uk/enable-your-research). This repository only contains the downstream analysis code; it does not distribute any UK Biobank data.

## Requirements

- Python 3.9+
- `pandas`, `numpy`, `matplotlib`, `seaborn`
- Access to the UK Biobank Research Analysis Platform (RAP) with the `dx` command-line toolkit configured, **or** local copies of the CSV files listed above placed in the notebook's working directory

Install the Python dependencies with:

```bash
pip install pandas numpy matplotlib seaborn
```

## Usage

1. Ensure the input CSVs listed above are available (either via `dx download` on the UKB RAP, or copied locally — edit the "Download data" cell accordingly).
2. Open `Reproduce_ESC_journal_all_models.ipynb` and run all cells in order.
3. To add or remove a model from the comparison, edit the `datasets` dictionary (cell under "Create a list for different models") — the dictionary key becomes the model's display name in all figures, and must match a variable name defined in the data-loading cell.

## Outputs

The notebook produces two sets of five-panel figures (one panel per metric: `N_high-risk`, `CVD_high-risk`, `CVD_prevented`, `NNS`, `NNT`), each showing both the delta relative to QRISK (bottom axis) and the absolute value (top axis):

- Single-step model comparison vs QRISK baseline.
- Two-step target-screening comparison vs QRISK baseline.
- Combined figure overlaying the single-step "Threshold" scenario and the matched-count "No Needle-prioritised" scenario.

## Citation

If you use or reference this analysis, please cite the two source studies this work builds on:

> Oexner RR, Schmitt R, Kals M, et al. Multi-disease risk prediction models for population-scale personalised screening. *medRxiv* 2026.02.02.26345271. https://doi.org/10.64898/2026.02.02.26345271

> Ritchie SC, Jiang X, Pennells L, et al. Combined clinical, metabolomic, and polygenic scores for cardiovascular risk prediction. *European Heart Journal* 2025. https://doi.org/10.1093/eurheartj/ehaf947

*(Add your thesis citation/reference here once finalised.)*

## License

*(Add a license here, e.g. MIT for the code — note that this only applies to the analysis code, not to any UK Biobank data, which remains subject to UK Biobank's own data-access terms.)*

## Author / Contact

- **Author:** *(your name)*
- **Institution:** *(your university / department)*
- **Supervisor:** *(advisor name, if you'd like to credit them)*
- **Contact:** *(email or GitHub handle)*

## Acknowledgements

This research uses the UK Biobank Resource. Analytical framework adapted from Oexner et al. (medRxiv, 2026) and Ritchie et al. (*European Heart Journal*, 2025); see Citation above.

