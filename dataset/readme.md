# Dataset Guide: eCommerce Events History in Cosmetics Shop

## Source

Kaggle dataset **eCommerce Events History in Cosmetics Shop**, published by `mkechinov`:
<https://www.kaggle.com/datasets/mkechinov/ecommerce-events-history-in-cosmetics-shop>

Check the licence and terms on the Kaggle page before redistributing the data. The CSV files are **not** included in this repository because of their size; download them from Kaggle.

## What the data is

Each row is one **user event** in an online cosmetics shop: a user viewing a product, adding it to the cart, removing it from the cart, or buying it. The data is event-level (a clickstream log), not order-level, so a single purchase of several items appears as several rows.

## Scope used in this project

| Item | Value |
|------|------:|
| Files | 5 monthly CSV files |
| Period | 1 October 2019 to 29 February 2020 |
| **Total rows** | **20,692,840** |
| Columns loaded by the notebook | 8 raw columns |
| Columns in the working table | 9 (8 raw columns minus `user_session`, plus 2 derived) |
| Unique users | 1,639,358 |
| Unique products | 54,571 |
| Unique sessions | 4,535,942 |
| Distinct `category_code` values | 12 |
| Distinct brands | 273 |

### Rows per file

| File | Rows | Share | First event | Last event | Active days |
|------|-----:|------:|-------------|------------|------------:|
| `2019-Oct.csv` | 4,102,283 | 19.8% | 2019-10-01 00:00:00 | 2019-10-31 23:59:54 | 31 |
| `2019-Nov.csv` | 4,635,837 | 22.4% | 2019-11-01 00:00:02 | 2019-11-30 23:59:58 | 30 |
| `2019-Dec.csv` | 3,533,286 | 17.1% | 2019-12-01 00:00:00 | 2019-12-31 23:59:57 | 31 |
| `2020-Jan.csv` | 4,264,752 | 20.6% | 2020-01-01 00:00:00 | 2020-01-31 23:59:58 | 31 |
| `2020-Feb.csv` | 4,156,682 | 20.1% | 2020-02-01 00:00:01 | 2020-02-29 23:59:59 | 29 |

Every month is complete (active days equal calendar days), and no row has a timestamp outside its file's month.

## Columns

The notebook reads eight columns. The Kaggle page also documents a ninth, `category_id`, which the notebook does not load. To confirm the exact header of your copy, run `pd.read_csv("2020-Jan.csv", nrows=0).columns`.

| Column | Type in notebook | Meaning | Missing | Notes |
|--------|------------------|---------|--------:|-------|
| `event_time` | datetime | Time of the event in UTC, to the second. | 0% | Parsed as UTC and stored without timezone. |
| `event_type` | category | Kind of event. | 0% | Four values: `view`, `cart`, `remove_from_cart`, `purchase`. See the table below. |
| `product_id` | int64 | Identifier of the product. | 0% | 54,571 distinct products. |
| `category_id` | not loaded | Numeric identifier of the product category (Kaggle documentation). | not checked | Unused: the textual `category_code` is used instead. |
| `category_code` | category | Category path such as `appliances.environment.vacuum`, with levels separated by dots. | **98.29%** | Only 12 distinct values. Almost always empty, so category analysis covers a small subset. |
| `brand` | category | Brand name in lower case (for example `runail`). | **42.32%** | 273 distinct brands. |
| `price` | float64 | Price of the product at the time of the event. | 0% | Currency is not stated in the files. Contains zero and negative values (see quality notes). |
| `user_id` | int64 | Identifier of the user. | 0% | 1,639,358 distinct users. The funnel counts unique `user_id` values. |
| `user_session` | text, then dropped | Identifier of the user's session. | 0% | Replaced in memory by `session_id` (a 64-bit hash). |

### Event types

| `event_type` | Events | Share | Meaning (interpretation) |
|--------------|-------:|------:|--------------------------|
| `view` | 9,657,821 | 46.67% | A user looked at a product. |
| `cart` | 5,768,333 | 27.88% | A user added a product to the cart. |
| `remove_from_cart` | 3,979,679 | 19.23% | A user removed a product from the cart. Not a funnel stage, so it is excluded from the funnel. |
| `purchase` | 1,287,007 | 6.22% | A user bought a product. |

The mix is similar every month: views 44.8% to 48.9%, carts 26.2% to 30.0%, removals 18.6% to 20.0%, purchases 5.8% to 7.0%.

## Derived columns created by the notebook

| Column | Definition |
|--------|------------|
| `month` | Month label (`2019-10` to `2020-02`) taken from the file name. |
| `session_id` | 64-bit hash of `user_session`, used to group purchases into baskets. |
| `category_main` | The first level of `category_code` (for example `appliances`). Six distinct values. Used only in the reconciliation section. |

## Data quality notes

| Issue | Size | Handling |
|-------|-----:|----------|
| `category_code` missing | 20,339,246 rows (98.29%) | Kept as missing. Category rankings exclude these rows. |
| `brand` missing | 8,757,117 rows (42.32%) | Kept as missing. Brand rankings exclude these rows. 42.7% of purchases have no brand. |
| Exact duplicate rows | 1,109,098 rows (5.36%) | Kept. The funnel counts unique users, so duplicates cannot inflate it. |
| Price of zero or below | 104,288 rows (0.50%) | Kept and reported. Negative prices also appear among purchases. They need a business explanation before revenue figures are used. |
| File month vs timestamp month | 0 mismatches | No action needed. |
| Missing `event_time`, `product_id`, `price`, `user_id` | 0 rows | None. |

## Purchase price snapshot

From the 1,287,007 purchase events: first quartile 1.59, median 3.00, third quartile 5.40, mean 4.93, maximum 327.78. Across 155,617 purchasing sessions, the mean basket value is 40.79 and the median 29.21.

## Things to know before using this data

- **Event rows are not orders.** Counting rows overstates people. Use unique `user_id` for people-level metrics.
- **Users are identified by `user_id` only.** The same person may have several IDs, and one ID may be shared.
- **The window is five months.** Behaviour before October 2019 and after February 2020 is not observed.
- **There is no information** about stock, promotions, traffic source, page content or order value beyond the event price. Explanations for conversion changes need those sources.

