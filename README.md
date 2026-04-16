# CBI — Cardiometabolic Burden Index in Critically Ill Cardiovascular Patients

**A NHANES-Anchored Cardiometabolic Burden Index and 28-Day Mortality in Critically Ill Cardiovascular Patients: A Dual ICU Cohort Study in MIMIC-IV and eICU-CRD**

Code, SQL extraction scripts, NHANES anchor parameters, and aggregate results for the analysis described in the accompanying manuscript.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.XXXXXXX.svg)](https://doi.org/10.5281/zenodo.XXXXXXX)

---

## Authors

Hasan Burak İşleyen, MD¹\*; Sevil Tuğrul Yavuz, MD²; Sercan Bulut, MD³; Fatih Kızkapan, MD³; Cevahir Alioğlu, MD⁴; Ali Arda Sözen, MD⁴; Mahsa Khanmohammadi, MD⁴

¹ Department of Cardiology, Nişantaşı University Faculty of Medicine, Istanbul, Turkey
² Department of Cardiology, Başakşehir Çam and Sakura City Hospital, Istanbul, Turkey
³ Department of Cardiology, Bağcılar Training and Research Hospital, Istanbul, Turkey
⁴ Department of Cardiology, Bezmialem Vakıf University Faculty of Medicine Hospital, Istanbul, Turkey

\*Corresponding author: hasanburak.isleyen@nisantasi.edu.tr | ORCID [0000-0002-4400-9715](https://orcid.org/0000-0002-4400-9715)

---

## ⚠️ Data-access policy

**No patient-level MIMIC-IV or eICU-CRD data are redistributed in this repository.** Both databases are credentialed-access resources under PhysioNet Data Use Agreements. To reproduce the full pipeline you must independently obtain access, as follows:

1. Complete the required CITI training: **"Data or Specimens Only Research"**, available at [citiprogram.org](https://www.citiprogram.org/).
2. Register a credentialed PhysioNet account at [physionet.org](https://physionet.org/register/).
3. Sign the Data Use Agreements for:
   - MIMIC-IV v3.1 → https://physionet.org/content/mimiciv/3.1/
   - eICU Collaborative Research Database v2.0 → https://physionet.org/content/eicu-crd/2.0/
4. Enable BigQuery access on physionet-data (both datasets mirror there).

**NHANES 2015–2018** files are public and freely available at https://wwwn.cdc.gov/Nchs/Nhanes/.

Aggregate summary statistics (baseline tables, hazard ratio point estimates, AUROC values, figure data) derived from the credentialed data are shared in `results/` in non-disclosive form in accordance with the PhysioNet DUA.

---

## Repository contents

```
.
├── README.md                         ← this file
├── LICENSE                           ← MIT license
├── CITATION.cff                      ← citation metadata (for Zenodo)
├── requirements.txt                  ← Python dependencies
├── sql/                              ← BigQuery extraction scripts
│   ├── mimic_cardiovascular_cohort.sql
│   ├── eicu_cardiovascular_cohort.sql
│   ├── mimic_labs_glucose_nlr_egfr.sql
│   └── eicu_labs_glucose_nlr_egfr.sql
├── analysis/                         ← Python analysis pipeline
│   ├── 01_nhanes_anchor.py           ← compute NHANES reference parameters
│   ├── 02_mimic_preprocess.py        ← MIMIC preprocessing and CBI scoring
│   ├── 03_eicu_preprocess.py         ← eICU preprocessing and CBI scoring
│   ├── 04_cox_primary.py             ← Model 2 primary and Model 3 secondary
│   ├── 05_discrimination.py          ← AUROC, IDI, NRI, calibration, DCA
│   ├── 06_spline_rcs.py              ← restricted cubic splines
│   ├── 07_subgroup_forest.py         ← 8 prespecified subgroup analyses
│   ├── 08_sensitivity.py             ← LOO, MICE, cause-specific Cox,
│   │                                   statin add-back, in-hospital harmonization
│   └── 09_pooled_interaction.py      ← cohort-stratified Cox + interaction test
├── results/                          ← aggregate outputs only (no PHI)
│   ├── all_results.json              ← full results registry
│   ├── nhanes_anchor.json            ← NHANES 2015-2018 reference parameters
│   ├── table1_baseline.csv           ← baseline characteristics (aggregate)
│   ├── table2_cox_sequential.csv
│   ├── table3_head_to_head.csv
│   └── subgroup_forest.csv
├── figures/                          ← publication-quality PNGs (300 dpi)
│   ├── Figure1_flowchart.png
│   ├── Figure2_KM_by_tertile.png
│   ├── Figure3_RCS_dose_response.png
│   ├── Figure4_subgroup_forest.png
│   ├── Figure5_calibration_DCA.png
│   └── FigureS1_leave_one_out.png
└── supp/                             ← supplementary methods detail
    └── mimic_nlr_unit_disambiguation.md
```

---

## How to reproduce

### 1. Environment setup

```bash
git clone https://github.com/drman44/cbi-dual-cohort.git
cd cbi-dual-cohort
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

### 2. NHANES anchor (public data, ~5 min)

```bash
python analysis/01_nhanes_anchor.py
# → writes results/nhanes_anchor.json
```

### 3. Credentialed data extraction (BigQuery)

After completing the PhysioNet credentialing steps above, set up Google Cloud authentication:

```bash
gcloud auth application-default login
```

Run each SQL script against `physionet-data.mimiciv_3_1_*` and `physionet-data.eicu_crd_v2_0_*`. The `.sql` files in `sql/` can be executed through the BigQuery console or via `bq query`. Export each result to a local CSV named in the analysis script headers.

### 4. Analysis pipeline

```bash
python analysis/02_mimic_preprocess.py
python analysis/03_eicu_preprocess.py
python analysis/04_cox_primary.py
python analysis/05_discrimination.py
python analysis/06_spline_rcs.py
python analysis/07_subgroup_forest.py
python analysis/08_sensitivity.py
python analysis/09_pooled_interaction.py
```

Each script reads the preprocessed analytic tables and writes results to `results/`. Figures are regenerated in `figures/`.

---

## Key results

| Specification | MIMIC-IV HR (95% CI) | eICU-CRD HR (95% CI) |
|---|---|---|
| Model 2 (primary) | **1.457 (1.403–1.512)** | **1.342 (1.277–1.409)** |
| Model 3 (secondary) | 1.418 (1.363–1.474) | 1.083 (1.026–1.143) |
| Pooled cohort-stratified | — | **1.421 (1.376–1.469)** on 17,905 patients |

28-day mortality: 10.01% (MIMIC-IV) and 14.62% (eICU-CRD). Third-versus-first tertile HR: 7.28 (MIMIC-IV) and 4.59 (eICU-CRD). ΔAUROC when added to the 12-covariate Model 3 baseline: +0.045 (MIMIC-IV) and +0.002 (eICU-CRD).

Head-to-head Model 2 AUROC comparison confirmed CBI exceeded every single component in both cohorts.

---

## Citation

If you use this code, please cite:

```
İşleyen HB, Tuğrul Yavuz S, Bulut S, Kızkapan F, Alioğlu C, Sözen AA, Khanmohammadi M.
A NHANES-Anchored Cardiometabolic Burden Index and 28-Day Mortality in Critically Ill
Cardiovascular Patients: A Dual ICU Cohort Study in MIMIC-IV and eICU-CRD.
[Journal, DOI when assigned].
```

BibTeX entry provided in `CITATION.cff`.

---

## License

This code is released under the [MIT License](LICENSE). Patient-level data from MIMIC-IV and eICU-CRD remain under their respective PhysioNet Data Use Agreements and are not covered by this license.

## Contact

Issues and questions: open a GitHub Issue or email the corresponding author.
