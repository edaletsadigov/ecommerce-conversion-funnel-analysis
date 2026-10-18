# E-Commerce Funnel Analysis: View → Cart → Purchase

A user-level conversion funnel built from 20.7 million raw events of an online cosmetics shop (October 2019 to February 2020). The project measures where users drop off, compares January with February 2020, and turns the findings into recommendations for a product or growth team.

**Stack:** Python, pandas, NumPy, matplotlib, seaborn.

## Headline results

| Question | Answer | Context |
|----------|--------|---------|
| How many viewers end up buying? | **6.6%** (104,757 of 1,597,754) | 22.4% of viewers add to the cart; 29.3% of carters buy |
| Did conversion improve from January to February? | **No.** 6.61% → 6.33% (-0.28 pp, -4.2% relative) | Dec → Jan moved -0.01 pp; the largest monthly move was Oct → Nov (+2.14 pp) |
| How long do buyers take? | **Two groups.** 36.8% buy within 1 hour, 56.2% within 24 hours | Median 8.2 h, but the late group takes days to weeks |
| Which category converts best? | `apparel.glove`, 19.4% | Only 2.4% of views carry a category, so this covers a small subset |
| How concentrated are sales by brand? | Top 5 brands = 25.1% of all purchases | 42.7% of purchases have no brand |
| Where is demand lost? | 418 popular products convert below 6.7% | They take 17.8% of product views |
| Are there price outliers? | 7.04% of purchases are IQR outliers | They carry 34.9% of revenue |

## Repository structure

```
ecommerce-funnel-analysis/
├── README.md                               <- you are here
├── Ecommerce_Funnel_Analysis_v2.ipynb      <- executed notebook with outputs
├── charts/                                 <- 8 PNG charts
└── docs/
    ├── README_CHARTS.md                    <- every chart explained
    ├── README_NOTEBOOK.md                  <- libraries, design decisions, how to run
    └── README_DATASET.md                   <- data description and quality notes
```

## Data and method in brief

- **Data:** five monthly files, 20,692,840 events, 1,639,358 users, 54,571 products. Details in [README DATASET](dataset/readme.md).
- **Funnel:** view → cart → purchase, counted as **unique users**. The default (strict) definition requires a user to pass each previous stage. The loose definition (each stage counted independently) gives almost the same answer: 24.9% / 27.7% / 6.9% against 22.4% / 29.3% / 6.6%.
- **Event-level vs user-level:** ratios of event counts (view→cart 59.7%) are much higher than user ratios (22.4%) because a few users generate many events. Only user-level rates are used for conclusions.
- **Statistics:** two-proportion z-tests with Bonferroni correction, each paired with an effect size. See [README NOTEBOOK](notebook/readme.md).

---

## Insights

### Insight 1: Both funnel steps leak, and the cart is where the intent is lost

![Overall funnel](charts/01_overall_funnel.png)

**Evidence.** 77.6% of viewers (about 1.24 million) never add a product to the cart, and 70.7% of carters never buy. Removals are also frequent: 3,979,679 `remove_from_cart` events against 5,768,333 `cart` events, so for every 100 additions there are about 69 removals (event counts, not users).

**Interpretation.** The view→cart step loses more users, but the cart→purchase step is where users have already shown intent, so it is the more direct place to recover sales. As arithmetic, not a forecast: +1 percentage point at cart→purchase adds about 3,580 buyers, and +1 percentage point at view→cart adds about 15,980 carters, or roughly 4,680 buyers at the current cart conversion.

**Action.** Investigate why carts are removed and abandoned (shipping cost, price, checkout friction), and test cart-recovery messages.
**Confidence.** Medium. User-level, five months, no information on the reasons behind removals.

Chart analysis: [Chart 01](docs/README_CHARTS.md#chart-01-overall-funnel).

### Insight 2: Buyers decide in two very different ways

![Time to purchase](charts/07_time_to_purchase.png)

**Evidence.** Among 102,237 converting users, 36.8% buy within one hour of their first view and 56.2% within 24 hours. The slowest quarter takes about 264 hours (11 days) or more, and the slowest tenth about 43 days or more. The distribution has two clusters: one peaking at roughly 20 to 30 minutes and a broad one from days to weeks.

**Interpretation.** The median (8.2 hours) sits in the gap between the clusters and describes neither. Quoting one "average time to purchase" would be misleading. The two groups most likely behave differently: quick same-session buyers and users who return after comparing or waiting.

**Action.** Use two tactics: in-session nudges for the fast group, and reminders or remarketing spread over days and weeks for the slow group. Measure each separately.
**Confidence.** Medium. The reasons for the two groups are inferred from timing only.

Chart analysis: [Chart 07](docs/README_CHARTS.md#chart-07-time-to-purchase).

### Insight 3: January → February did not improve conversion, and the change is practically negligible

![Monthly funnel trend](charts/03_monthly_funnel_trend.png)

![January vs February](charts/04_jan_vs_feb_conversion.png)

**Evidence.** View→purchase moved from 6.61% to 6.33% (-0.28 pp). The decline comes from cart→purchase (32.38% → 30.63%); view→cart rose slightly (20.41% → 20.65%). All three differences are statistically detectable because each month has about 380,000 viewers, but every effect size is negligible (|Cohen's h| of 0.04 or less). For context, December → January moved -0.01 pp (p = 0.88). The one large move in the data is October → November (+2.14 pp, +35% relative): view→cart fell (31.7% → 23.7%) while cart→purchase nearly doubled (19.4% → 35.1%).

**Interpretation.** January to February is within normal month-to-month movement. November is the unusual month. It contains Black Friday week, so a seasonal or promotional effect is plausible but untested, because the data has no campaign information.

**Action.** Do not report February as an improvement. Track monthly user-level conversion with a stated minimum meaningful change (for example, a relative change of several percent), and use controlled A/B tests to attribute causes.
**Confidence.** Medium for the direction; low for any cause. 15.8% of February viewers were also January viewers, so the samples are not independent.

Chart analysis: [Chart 03](docs/README_CHARTS.md#chart-03-monthly-funnel-trend), [Chart 04](docs/README_CHARTS.md#chart-04-january-vs-february).

### Insight 4: Category and brand data are too incomplete to steer on, but they reveal clear mismatches

![Conversion by category](charts/02_category_conversion.png)

![Top 10 brands](charts/05_top_brands.png)

**Evidence.**

- `category_code` is missing for 98.3% of events. Within the tagged 2.4% of views, view→purchase ranges from 1.5% (`furniture.living_room.cabinet`) to 19.4% (`apparel.glove`), a 13-fold gap.
- `accessories.bag` has a healthy cart→purchase rate (29.8%) but only 5.2% of viewers add it to the cart, so its problem is before the cart.
- The top 5 brands make 25.1% of all purchases; `runail` alone makes 8.7%. 42.7% of purchases have no brand.
- `estel` has the fourth-largest audience of the top ten (92,359 viewers) but the lowest conversion (7.3%). `bpw.style` (17.8%) and `freedecor` (16.8%) convert best on much smaller audiences.
- Purchase count is not revenue: `uno` earns 10.84 per purchase event, `freedecor` 1.30.

**Interpretation.** The category ranking describes a small, mostly non-cosmetic subset, so it cannot guide assortment decisions. The brand comparison is more reliable, but 42.7% of purchases are invisible to it.

**Action.** Fix `category_code` and `brand` tracking first. Then review `estel` product pages, and test whether more visibility for `bpw.style` and `freedecor` keeps their conversion at larger scale.
**Confidence.** Low for categories (coverage), medium for brands.

Chart analysis: [Chart 02](docs/README_CHARTS.md#chart-02-conversion-by-category), [Chart 05](docs/README_CHARTS.md#chart-05-top-10-brands).

### Insight 5: A small group of popular products wastes a large share of attention

![High-view, low-conversion products](charts/08_high_view_low_conversion_products.png)

**Evidence.** 418 products (3.0% of the 13,731 with at least 100 viewers) are in the top 5% by viewers yet convert below the 6.66% benchmark. They take 17.8% of all product views. At benchmark conversion they would add about 41,335 purchases. The biggest gaps are product 5877456 (15,150 viewers, 0.34%, median price 122.22) and product 5808664 (10,660 viewers, 0.33%, median price 6.37).

**Interpretation.** The two leading examples differ in price, so the causes differ: a high price is a plausible barrier for the first; for the cheap second product stock, images or reviews are more likely. These are hypotheses. 41,335 is an upper bound that ranks the effort, not a forecast.

**Action.** Audit the top 20 flagged products first (price against similar items, availability, images, reviews, page speed) and run page-level A/B tests on the best candidates.
**Confidence.** Medium. The data has no stock or content information, and low conversion can reflect comparison browsing.

Chart analysis: [Chart 08](docs/README_CHARTS.md#chart-08-high-view-low-conversion-products).

### Insight 6: A few high-priced purchases carry a third of revenue

![Price outliers](charts/06_price_outliers.png)

**Evidence.** The median purchase price is 3.00 and the mean 4.93. 90,657 purchases (7.04%) lie above the IQR fence of 11.12 and generate 34.9% of revenue. The highest-priced rows are repeated purchases of the same products at the same price (for example product 5906221 at 327.78, five times between December and February), so they look like real premium items, not entry errors. Mean basket value is 40.79 against a median of 29.21.

**Interpretation.** Revenue depends on a thin premium tail, so averages overstate the typical order and conversion of premium products deserves separate tracking. Some purchases have negative prices, and 104,288 rows (0.5%) have a price of zero or below. These need an explanation before revenue reporting.

**Action.** Report medians next to means, track premium products separately, and investigate the zero and negative prices (refunds, free items or logging errors).
**Confidence.** Medium.

Chart analysis: [Chart 06](docs/README_CHARTS.md#chart-06-price-outliers).

---

## Recommendations

| Priority | Recommendation | Evidence | How to measure success |
|----------|----------------|----------|------------------------|
| 1 | Fix tracking of `category_code` (98.3% missing) and `brand` (42.3% missing) | Insight 4 | Share of events with a category and brand above 90% |
| 2 | Reduce cart loss: study removals, test cart-recovery messages and checkout friction | Insight 1: 70.7% of carters do not buy; 69 removals per 100 additions | Cart→purchase rate above the current 29.3% in an A/B test |
| 3 | Split retargeting by decision speed: in-session nudges plus reminders over days and weeks | Insight 2 | Conversion of each group measured separately |
| 4 | Audit the 418 high-view, low-conversion products, starting with the largest gaps | Insight 5 | Product conversion approaching the 6.66% benchmark |
| 5 | Review `estel` and test more visibility for `bpw.style` and `freedecor` | Insight 4 | Brand conversion and revenue per viewer |
| 6 | Stop reporting February as an improvement; track monthly user-level conversion with a minimum meaningful change | Insight 3 | Pre-registered A/B tests for changes |
| 7 | Clean prices of zero or below and report medians next to means | Insight 6 | Zero negative-priced purchases in reporting tables |

## Important result: the reference notes do not reproduce

The task came with reference figures. Computed on these files (January and February only, the same scope), none of them match:

| Metric | Reference note | Computed |
|--------|---------------:|---------:|
| Rows, January / February | ~2.6M / ~1.7M | 4,264,752 / 4,156,682 |
| View / cart / purchase share of events | 85.1% / 8.7% / 6.2% | 58.8% / 33.8% / 7.4% |
| View→cart, event level | 10.2% | 57.6% |
| Cart→purchase, event level | 71.4% | 22.0% |
| View→purchase, event level | 7.3% | 12.7% |
| February minus January conversion | +0.8 pp | -0.56 pp (events), -0.28 pp (users) |
| Best category (`accessories`) | 12.1% | 1.8% |
| Top 5 brands, share of purchases | 34% | 25.3% |
| Average / maximum basket | 26.8 / 2,140 | 41.03 / 1,566.81 |

The notes were probably produced from another dataset, file version or definition. This project therefore uses **only values computed from the files** and does not rely on the notes. The full table is in section 8 of the notebook.

## Limitations

- **Window:** October 2019 to February 2020 only. Users who viewed before October and bought later appear as purchasers without a view (4.8% of purchasers).
- **Window length:** the overall funnel spans five months and single-month funnels one month, so their rates cannot be compared with each other.
- **Users are IDs, not people.** Users active in two months are counted in both monthly funnels.
- **Missing attributes:** categories are 98.3% missing and brands 42.3% missing, so category and brand findings describe the tagged part only.
- **No causation:** every difference between months, categories or brands is an association. Causes need controlled experiments, and the data has no promotion, stock or traffic-source information.
- **Prices:** the currency is not stated, and negative prices exist.

## Reproduce

1. Download the five CSV files from [Kaggle](https://www.kaggle.com/datasets/mkechinov/ecommerce-events-history-in-cosmetics-shop).
2. Place them next to `Ecommerce_Funnel_Analysis_v2.ipynb`.
3. `pip install pandas numpy matplotlib seaborn jupyter`, then run all cells.

Full instructions, including a quick dry-run option, are in [docs/README_NOTEBOOK.md](docs/README_NOTEBOOK.md).
