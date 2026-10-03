# AD-MultiPathology

Patient-level quantification of four hippocampal proteinopathies, the notebook that produced the shape tests, and the stored result tables and figures.

https://github.com/Gutman-Lab/AD-MultiPathology

This deposit does not include whole-slide images, model weights, or the manuscript file. The numbers below are the stored outputs. Opening a CSV does not rerun detection.

## Layout

```
data/        patient tables and the six pre-specified test tables
notebooks/   abeta_analysis-final.ipynb
results/     figures and the summary tables drawn from data/
```

Start with `data/multistain_burdens.csv` (one row per patient) and `results/figures/fig1_pipeline.png` (what was measured). Use the test CSVs in `data/` for the axis-ratio results. Use `results/tables/` when you want the same numbers already rounded into the summary tables.

## What was measured

Each quantified row is one autopsy patient with a hippocampal block. Four DAB stains were measured separately: amyloid-β, tau, α-synuclein, and TDP-43.

**Burden** is footprint area percent: `100 × stained object area / slide footprint`. Column names are `abeta_pct`, `tau_pct`, `syn_pct`, and `tdp_pct`. This is not gray-matter-normalized burden. A value of `0.16` means 0.16 percent of the slide footprint, not 16 percent.

**Axis ratio** is minor axis divided by major axis. The value is stored as the HDF5 field `elongation`. A value near 1 is rounder. A smaller value is more elongated.

**Axis-ratio entropy** is Shannon entropy, in nats, of that axis-ratio distribution on 10 equal bins of the interval [0, 1].

Object counts in the burden table are `abeta_n`, `tau_n`, `syn_n`, and `tdp_n`.

## Cohort files

### `data/multistain_burdens.csv`

87 patients and 13 columns.

| Column | Meaning |
| --- | --- |
| `patient_id` | Case identifier |
| `block` | Tissue block identifier |
| `Braak`, `CERAD`, `Thal` | Neuropathological scores as stored. CERAD value `5` is not recoded. |
| `abeta_n`, `tau_n`, `syn_n`, `tdp_n` | Detected object counts |
| `abeta_pct`, `tau_pct`, `syn_pct`, `tdp_pct` | Footprint area percent |

TDP-43 burden is missing for 2 of the 87 patients. Amyloid-β has no zero-burden row. Tau, α-synuclein, and TDP-43 contain some exact zeros. Those zeros were treated as missing in the co-burden correlations reported in `results/tables/table_coburden_spearman.csv`. Leaving the zeros in changes those correlations, so use that table rather than recomputing Spearman on the raw zeros.

Summary of the same 87 rows is in `results/tables/table_ii_burden.csv`:

| Stain | Non-missing | Exact zeros | Median % | Min % | Max % |
| --- | ---: | ---: | ---: | ---: | ---: |
| Amyloid-β | 87 | 0 | 0.1606 | 0.0001 | 0.9512 |
| Tau | 87 | 1 | 0.5114 | 0 | 2.9608 |
| α-synuclein | 87 | 5 | 0.0168 | 0 | 3.2009 |
| TDP-43 | 85 | 4 | 0.0165 | 0 | 2.0836 |

### `data/multistain_analysis_table.csv`

The same 87 patients, plus the columns used when a shape summary was residualized. `Age_death` is the age used for that residualization: 84 non-missing values. Extra columns are `_k`, `frac_diffuse`, `frac_compact`, `frac_elongated`, `mean_eccentricity`, `mean_elongation`, and `LBD_bin`.

Do not replace `Age_death` with the age column from the demographics workbook. The two age fields agree on the patients who have both, but the residualization used this analysis-table column.

### `data/abeta_patient_demographics.xlsx`

Bank workbook, 233 cases on the `Patient Table` sheet. Other sheets are `Summary`, `Neuropath Scores`, `ApoE Genotype`, and `Group × Sex × Race`.

Patient Table columns: `#`, `Case Number`, `Emory Number`, `Diagnosis Group`, `Primary Neuropathologic Diagnosis`, `Clin Dx`, `Sex`, `Race`, `Age at Death/Bx`, `Age at Onset`, `Duration (years)`, `PMI (hr)`, `Braak Stage`, `CERAD`, `Thal`, `ABC`, `ApoE`, `Family Hx`.

There is no MMSE, CDR, or other cognitive-test column. This file cannot be used to test a correlation with cognitive testing.

The 233-row bank is not the quantified cohort. To match a burden row, take the token of the form `Eyy-nn` from `Case Number` and from `patient_id`. After one row per demographics identifier, 83 of the 87 burden rows match, and those 83 are labeled `AD/MCI` in `Diagnosis Group`. Four burden identifiers do not match an `Eyy-nn` case number: `OS-240715`, `OS-160608`, `OS_170124`, `OS_170518`.

The workbook contains case numbers, Emory numbers, clinical diagnosis, race, family history, and ApoE. Review those fields before a public release.

## How to read a test row

Primary tests residualize the shape summary on its own stain’s burden, age at death, and Braak stage, then correlate that residual with the named outcome. Ranks are average ranks. The correlation is Pearson on the residualized ranks, with a *t* reference distribution. Bootstrap confidence intervals use 10,000 resamples, seed 42, and percentile limits.

Within each family of six tests, `q_bh` is the Benjamini–Hochberg adjusted *p* value. A test is counted as support only when `q_bh < 0.05` and the 95% interval excludes zero. A confidence interval that excludes zero at an unadjusted *p* near 0.05 is not support.

Columns shared by the primary and sensitivity files:

| Column | Meaning |
| --- | --- |
| `role` | `primary` or the sensitivity rerun |
| `test_id` | Short label inside that file (`P1`–`P6` or `D1`–`D6`) |
| `test` | Full comparison |
| `n`, `rho`, `p`, `df` | Sample size, correlation, unadjusted *p*, degrees of freedom |
| `k_covariates` | Number of covariates removed. 3 for a shape-to-burden test. 4 for a shape-to-shape test. |
| `ci95_low`, `ci95_high` | Bootstrap percentile interval |
| `bootstrap_resamples` | 10000 |
| `method` | Rank residual correlation, as above |
| `min_instances` | A patient needs at least 10 objects on that stain |
| `seed` | 42 |
| `q_bh` | BH-adjusted *p* inside that family of six |

### Median axis ratio

| File | Role |
| --- | --- |
| `data/s15_primary_analysis.csv` | Six pre-specified tests. This is the headline family. |
| `data/s15_sensitivity.csv` | Same six tests after exact zeros are set to missing. Not the headline. |
| `data/s15_validation.csv` | Same-stain Spearman correlation of median axis ratio with that stain’s own burden. Not part of the six-test family and not BH-adjusted. |
| `results/tables/table_iii_s15_primary.csv` | Summary of the primary file |
| `results/figures/fig4_median_axis_ratio.png` | Primary intervals |
| `results/figures/fig_median_axis_ratio.png` | Distribution figure written with the tests |

The primary family does not meet the support rule. The smallest `q_bh` is 0.284, on α-synuclein residual axis ratio versus TDP-43 burden (`n = 76`, `rho = -0.228`, unadjusted `p = 0.052`). That interval excludes zero. It still does not meet `q_bh < 0.05`.

The sensitivity file is a rerun, not a replacement. Its smallest `q_bh` is 0.051.

`s15_validation.csv` shows that median axis ratio tracks own-stain burden for tau, α-synuclein, and TDP-43. That is why the primary tests residualize on own burden before any cross-stain comparison. The validation `note` column says these rows are not the primary test.

### Axis-ratio entropy

| File | Role |
| --- | --- |
| `data/s16_primary_analysis.csv` | Six pre-specified entropy tests |
| `data/s16_sensitivity.csv` | Same six tests with exact zeros set to missing |
| `results/tables/table_iv_s16_primary.csv` | Summary of the primary file |
| `results/figures/fig5_axis_ratio_entropy.png` | Primary intervals |
| `results/figures/fig_axis_ratio_entropy_distribution.png` | Distribution figure written with the tests |

The entropy family does not meet the support rule. The smallest `q_bh` is 0.300, on α-synuclein entropy residual versus TDP-43 entropy residual (`n = 72`, `rho = 0.239`, unadjusted `p = 0.050`). That interval includes zero. The sensitivity file has the same smallest `q_bh`.

## Other result figures

| File | What it shows |
| --- | --- |
| `results/figures/fig1_pipeline.png` | Detection, object measurements, patient burden, then the rank residual |
| `results/figures/fig2_burden_distributions.png` | Footprint area percent for the four stains |
| `results/figures/fig3_coburden_spearman.png` | Co-burden Spearman correlations after exact zeros are removed |
| `results/tables/table_coburden_spearman.csv` | The three co-burden rows plotted in that figure |

Co-burden rows, with zeros removed:

| Comparison | n | rho | p |
| --- | ---: | ---: | ---: |
| Tau burden vs CERAD | 83 | 0.231 | 0.036 |
| Amyloid-β burden vs α-synuclein burden | 82 | 0.319 | 0.0035 |
| α-synuclein burden vs TDP-43 burden | 77 | 0.303 | 0.0073 |

## Notebook

`notebooks/abeta_analysis-final.ipynb` is the analysis notebook that wrote the median-axis-ratio and entropy CSV files. It is stored with its printed outputs. Those printed outputs and the CSV files are the record of the run.

The notebook’s path cells point at the machine where it was executed. They do not point at this repository. Re-running a cell will not find the whole-slide images from this folder, because the images are not deposited here. To check a reported coefficient, read the matching CSV instead of re-executing the notebook.

## Using the files

1. Read burdens from `data/multistain_burdens.csv`. Keep `patient_id` as a string.
2. Join age for a residualization check on `patient_id` to `data/multistain_analysis_table.csv`, and use `Age_death`.
3. Join bank sex, race, onset, duration, or diagnosis only through the `Eyy-nn` token described above. Drop duplicate demographics identifiers before the join. A many-to-many merge inflates the 87 rows.
4. Report a shape result from `s15_primary_analysis.csv` or `s16_primary_analysis.csv`. Require both `q_bh < 0.05` and a confidence interval that excludes zero.
5. Quote co-burden coefficients from `results/tables/table_coburden_spearman.csv`.
