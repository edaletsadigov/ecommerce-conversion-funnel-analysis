# Notebook Guide: `Ecommerce_Funnel_Analysis.ipynb`

This document explains what the notebook does, which libraries it uses and why, the main design decisions, and how to run it.

## What the notebook does

It turns raw event-level data (20,692,840 rows across five monthly CSV files) into a user-level conversion funnel (view → cart → purchase) and uses it to answer product and growth questions:

- where users drop off, overall and by category, brand and month
- whether conversion changed from January to February 2020, and whether that change is unusual
- how long converting users take to decide
- which heavily viewed products convert poorly
- whether purchase prices contain outliers

The executed notebook (with outputs) is included in this repository. The charts are exported to [`../charts`](../charts) and explained in [README CHARTS](charts/readme.md). The data is described in [README DATASET](dataset/readme.md).

## Notebook structure

| Section | Content | Main output |
|---------|---------|-------------|
| Libraries and configuration | Imports and every tunable constant in one place | none |
| 1. Data Overview and Cleaning | Load five files, coverage per month, missing values, duplicates, event-type distribution, data-quality checks | tables |
| 2. Funnel Construction | Helper functions, overall funnel under two definitions, event-level vs user-level ratios | Chart 01 |
| 3. Conversion Breakdown by Category | Funnel inside each of the top 8 categories | Chart 02 |
| 4. Monthly Funnel and January vs February | Funnel per month, two-proportion z-tests, month-over-month baseline | Charts 03, 04 |
| 5. Top Brands | Top 10 brands by purchase events with their funnel | Chart 05 |
| 6. Price Outliers | IQR method on purchase prices, basket-level view | Chart 06 |
| 7. Bonus Analyses | Time from first view to first purchase; high-view, low-conversion products | Charts 07, 08 |
| 8. Reconciliation with the Reference Notes | Compares the task's reference figures with the computed values | table |
| 9. Final Summary | Five insight cards generated from the computed results, and limitations | text |

## Libraries and why they are used

The task requires pandas, matplotlib and seaborn. Everything else comes from the Python standard library or IPython, which ships with Jupyter.

| Library | Used for | Why this library |
|---------|----------|------------------|
| **pandas** | Reading the CSV files with explicit dtypes, concatenating months, `groupby` / `agg`, `crosstab`, datetime handling, categorical columns | The standard tool for tabular event data. Categorical dtypes keep 20.7M rows at about 890 MB in memory (reported by `df.info()`). |
| **numpy** | `intersect1d` and `setdiff1d` on arrays of user IDs, `where` and `clip` for index arithmetic | Funnel stages are sets of unique users. Set operations on sorted integer arrays are much faster than Python loops or row-wise `apply` at this size. |
| **matplotlib** | Figure and axes creation, layout, axis labels, titles, reference lines (`axvline`, `axhline`), log scales | Gives full control over each chart. Seaborn draws on matplotlib axes, so both are needed. |
| **seaborn** | `barplot`, `heatmap`, `lineplot`, `histplot`, `boxplot`, `scatterplot` | Concise, consistent statistical charts with a clean default theme. The heatmap suits the category × step rate matrix. |
| **math** (standard library) | `sqrt`, `asin`, `erfc` | Implements the two-proportion z-test, its p-value and Cohen's h without SciPy, which keeps the stack to what the task specifies. |
| **pathlib** (standard library) | `Path` objects for file locations | Paths work the same on Windows, macOS and Linux. |
| **IPython.display** | `display` and `Markdown` | Renders tables and the generated insight text inside the notebook. |

Not used on purpose: **SciPy** (the z-test is a few lines of `math`), **plotly** (the task specifies matplotlib and seaborn), and any row-wise `apply` or `iterrows` (vectorised pandas and numpy are used everywhere).

## Key design decisions

**Strict and loose funnels.** The task defines each stage as "at least one view / cart / purchase". Counted independently (loose), a purchase without an observed view still counts, and rates can exceed what a real funnel allows. The strict definition requires a user to have passed the previous stage (view ⊇ cart ⊇ purchase). The notebook computes both. The constant `FUNNEL_MODE` chooses which one feeds the charts; the default is `"strict"`. In this data the two differ by at most 2.5 percentage points.

**User level, not event level.** One user viewing 50 products is one funnel user, not 50. Event-level ratios are printed only for comparison with the reference notes.

**Memory-efficient loading.** Each file is read once with `usecols` and explicit dtypes. `event_type`, `category_code` and `brand` are `category`. The long `user_session` string is replaced by a 64-bit hash (`session_id`) that is exact enough for grouping and much smaller. A `month` label is added from the file name before concatenation.

**Honest statistics.**

- Month comparisons use two-proportion z-tests with the null hypothesis "the rate is the same in both months".
- Every p-value is paired with an effect size (Cohen's h), because with hundreds of thousands of users even tiny differences are significant.
- Bonferroni correction is applied: alpha / 3 for the three funnel steps, alpha / 4 for the four month-over-month tests.
- Users active in two months appear in both funnels, so samples are not fully independent. The notebook prints this overlap.

**Outliers are measured, not deleted.** IQR outliers are counted and their revenue share is reported. High prices were checked against repeated purchases of the same products and kept.

**Small-sample guards.** Rankings require at least `MIN_VIEWERS` (100) unique viewers.

## Configuration constants

All constants are in one cell near the top.

| Constant | Default | Meaning |
|----------|---------|---------|
| `DATA_DIR` | `Path(".")` | Folder containing the CSV files. By default the notebook expects them next to it. |
| `FILES` | five monthly files | Month label → file name. |
| `COMPARE_MONTHS` | `("2020-01", "2020-02")` | The two consecutive months compared in section 4. |
| `NOTE_MONTHS` | same as above | Months to which the reference notes in section 8 apply. |
| `USER_SAMPLE_PERCENT` | `100` | Keep a deterministic share of users (for example `20`) for a quick dry run. Counts shrink, rates stay comparable. |
| `FUNNEL_MODE` | `"strict"` | `"strict"` or `"loose"` funnel definition. |
| `TOP_N_CATEGORIES` / `TOP_N_BRANDS` | `8` / `10` | Number of categories and brands ranked. |
| `MIN_VIEWERS` | `100` | Minimum unique viewers for a product or category to be ranked. |
| `VIEW_QUANTILE` | `0.95` | "High views" means the top 5% of eligible products by unique viewers. |
| `IQR_MULTIPLIER` | `1.5` | Fence width for outliers. |
| `ALPHA` | `0.05` | Significance level before Bonferroni adjustment. |

## Main functions

| Function | Purpose |
|----------|---------|
| `load_month` | Reads one monthly file with efficient dtypes and checks that required columns exist. |
| `concat_monthly` | Concatenates months while keeping categorical columns categorical. |
| `build_funnel` | Counts unique users per stage under both definitions and reports purchasers without a view or cart. |
| `funnel_rates` | Converts counts to step conversion rates for the chosen definition. |
| `funnel_by_group` | Runs the same funnel for every category, brand or other group. |
| `two_proportion_test` / `effect_label` | Z-test, p-value, Cohen's h and its plain-language label. |
| `summarise_hours` | Distribution summary of time-to-purchase. |

## How to run

1. Download the five monthly files from the Kaggle dataset *eCommerce Events History in Cosmetics Shop* (`2019-Oct.csv`, `2019-Nov.csv`, `2019-Dec.csv`, `2020-Jan.csv`, `2020-Feb.csv`). Link and column details are in [README_DATASET.md](dataset/readme.md).
2. Put the files in the same folder as the notebook, or change `DATA_DIR`.
3. Install the requirements: `pip install pandas numpy matplotlib seaborn jupyter`.
4. Open the notebook and run all cells from top to bottom.

The CSV files are large and are excluded from version control by `.gitignore`. The stored outputs in this repository come from a run on the full data with Python 3.14 and pandas 3.x. The code uses no random sampling, so repeated runs give identical results. Other pandas versions were not tested.

Loading and counting 20.7M rows takes a few minutes and several gigabytes of RAM at peak, because each CSV is read in full before the long session string is dropped. If memory is tight, set `USER_SAMPLE_PERCENT = 20` first.

## Known limitations of the notebook

- The chart saved as Chart 03 has its legend over the last cart→purchase point.
- Section 9's generated text labels the January to February change as "statistically supported". The effect is negligible in practical terms. The main [README](../README.md) states this more carefully.
- Section 3 ranks only categories with a `category_code`, which is 2.4% of view events.
- Section 8 shows that none of the reference figures reproduce on these files (see the main README).

