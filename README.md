# Reporting delay and calendar clustering in NUFORC public sighting reports

CSE558 Data Science, Monsoon 2026, IIIT-Delhi. Midterm project.

We parse and clean the raw NUFORC UFO sighting reports, split them 80/20 into an
exploration set and a confirmation set (seed 42), explore the exploration set, and run
hypothesis tests on the confirmation set. The two problem statements are:

- **Primary.** Among July reports, is the share that falls on the 4th larger than the share
  on other July days once weekday (weekend versus weekday) is held fixed?
- **Secondary.** Do reports filed more than 30 days after the sighting state longer
  durations than reports filed within 7 days?

## Hypothesis tests

All four tests use the confirmation set (28,249 reports) and a significance level of 0.05.
P-values are corrected for running four tests with Holm's method.

| # | Question | Test | Result |
|---|----------|------|--------|
| 1 | Is July 4th over-represented once weekday is held fixed? | permutation test within year and weekday/weekend groups | 5.6 times an ordinary July day (95% CI 4.1 to 7.2), p = 0.0001 |
| 2 | Do reports filed after 30 days state longer durations than reports filed within 7 days? | Mann-Whitney U | 1.21 times as long (95% CI 1.13 to 1.29), p = 1.3e-8 |
| 3 | Does duration rise steadily with delay, or only for very late reports? | Kruskal-Wallis, then Mann-Whitney between neighbouring delay bands | flat for the first year, then 1.49 times as long after a year, p = 4.1e-17 |
| 4 | Do July 4th reports describe a fireball more often than other July days? | two-proportion z-test | 14.1% against 8.9% (difference 5.2 points, 95% CI 1.7 to 8.7), p = 0.0005 |

All four reject the null hypothesis after the Holm correction. Tests 1 to 3 hold in every
check (without NUFORC hoax reports, with one report per city and night, without the -1 day
same-night reports). Test 4 does not hold once whole dates or whole years are resampled instead
of single reports (95% CI -1.5 to 11.0 and -0.3 to 10.4 points), so it is only suggestive.

For Test 2 the confidence interval of the ratio comes from a Welch t-test on log durations, so it
describes geometric means; the Mann-Whitney effect, P(late > early) = 0.525, has its own bootstrap
interval of 0.516 to 0.533.

## Data

- **File used:** `nuforc_reports.json` (141,261 records, 11 fields).
- **Download:** Kaggle, "UFO Sightings" (Arvidsson, 2023):
  https://www.kaggle.com/datasets/joebeachcapital/ufo-sightings
- **Reporting system:** National UFO Reporting Center (NUFORC), http://www.nuforc.org/
- **Scraper:** Timothy Renner, https://github.com/timothyrenner/nuforc_sightings_data

The dataset is not in this repository (about 224 MB). After downloading, place the JSON at:

```
data/nuforc_reports.json
```

To check that you have the same file (224,216,951 bytes), its SHA-256 should be
`F9406EF4E239829B275C1AD8687F423EC5D7F09D9CF030E177938EDDAD326750`:

```
certutil -hashfile data\nuforc_reports.json SHA256     (Windows)
sha256sum data/nuforc_reports.json                     (Linux / macOS)
```

We do not use the processed `nuforc_reports.csv` from the same download: it has already been
cleaned, which would skip the preparation step.

## Setup and run

Python 3.12.

```
git clone https://github.com/HardikGargg/dazzlingDataminers.git
cd dazzlingDataminers
python -m venv .venv
.venv\Scripts\activate          (Windows; on Linux / macOS: source .venv/bin/activate)
pip install -r requirements.txt
```

Place the data file as described above, then run the whole notebook from the command line:

```
jupyter nbconvert --to notebook --execute --inplace notebook.ipynb
```

or open `notebook.ipynb` and run all cells from top to bottom (Kernel > Restart & Run All).
A full run takes about two minutes and rewrites everything in `outputs/` and `figures/`.

## Reproducibility

- Random seed: `42`, set once at the top of the notebook.
- The exploration and confirmation splits are saved as lists of `report_link` values in
  `outputs/`, so the split can be rebuilt without re-drawing it.
- A fresh clone, a new virtual environment and the command above reproduce every file in
  `outputs/` and `figures/` byte for byte.

## Repository layout

```
notebook.ipynb     parsing, cleaning, encoding, split, EDA and tests
requirements.txt   pinned library versions
figures/           EDA plots used in the report
outputs/           tables written by the notebook
data/              raw dataset (not committed)
```

### `outputs/`

| File | Contents |
|------|----------|
| `before_summary.csv` | rows, columns, cells and duplicates of the raw file |
| `before_fields.csv` | missing values and distinct values for each raw field |
| `cleaning_log.csv` | every cleaning step and how many rows it affected |
| `size_check.csv` | rows, columns and cells before and after encoding |
| `explore_ids.csv` | `report_link` of the 112,998 reports in the exploration set |
| `confirm_ids.csv` | `report_link` of the 28,249 reports in the confirmation set |
| `split_check.csv` | July and delay-group counts in each set |
| `test3_steps.csv` | Test 3 comparisons between neighbouring delay bands |
| `test_results.csv` | statistic, p-value, estimate, confidence interval and Holm-adjusted p-value for each test |
| `robustness_checks.csv` | each test repeated without hoax reports, with one report per city and night, without -1 day reports, and with dates or years resampled |

### `figures/`

Each plot is saved as PDF (for the report) and PNG. All use the exploration set.

| File | Shows |
|------|-------|
| `fig_july_days` | reports on each day of July, July 4th highlighted |
| `fig_duration` | distribution of stated durations on a log scale |
| `fig_duration_by_delay` | stated duration for four reporting-delay bands |
| `fig_july_shapes` | shape shares on July 4th against other July days |

## Group

| Name | Roll no. | Email | Contribution |
|------|----------|-------|--------------|
| Manaswi Singh | 2023307 | manaswi23307@iiitd.ac.in | loading the raw data, before numbers, cleaning dates and reporting delays |
| Aryan Arora | 2024119 | aryan24119@iiitd.ac.in | cleaning durations, shapes, countries and junk rows; encoding and size check |
| Divyanshi | 2023209 | divyanshi23209@iiitd.ac.in | exploration / confirmation split, exploratory analysis and figures |
| Hardik Garg | 2024228 | hardik24228@iiitd.ac.in | hypotheses, Tests 1 to 4 and the alternative tests |
| Kanupriya Gupta | 2023269 | kanupriya23269@iiitd.ac.in | robustness checks, Holm correction, report and README |
