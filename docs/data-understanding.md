# Data Understanding

**CRISP-DM Phase 2**  
**Iteration 1 Deliverable**  
*Document version 1.0 — template, to be completed during iteration 1*

---

## Objective

To thoroughly understand the REES46 e-commerce clickstream dataset before any
modelling begins. This phase answers the question: **what does the data tell
us?** The findings here directly inform every decision made in data preparation
and modelling.

---

## 1. Dataset Overview

**Name:** eCommerce Behavior Data from a Multi-Category Store  
**Source:** Kaggle — published by REES46  
**URL:** https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store  
**Coverage:** October 2019 to April 2020  
**Raw size:** 233,460,662 rows  

**Schema:**

| Column | Data Type | Description |
|---|---|---|
| `event_time` | timestamp | Precise moment of the interaction (UTC) |
| `event_type` | string | view, cart, or purchase |
| `product_id` | integer | Unique product identifier |
| `category_id` | long | Category the product belongs to |
| `category_code` | string | Hierarchical category label — frequently missing |
| `brand` | string | Brand name — absent in some instances |
| `price` | float | Monetary value of the product |
| `user_id` | integer | Persistent identifier across sessions |
| `user_session` | string | Temporary session ID — resets after prolonged inactivity |

---

## 2. Data Quality Assessment

*To be completed in iteration 1*

### 2.1 Missing Values

| Feature | Number of nulls | Percentage | Handling plan |
|---|---|---|---|
| `category_code` | 47,496,567 | TBD | Exclude — redundant with `category_id` |
| `brand` | 29,948,661 | TBD | Encode missing as "empty" |
| All others | 0 | 0% | No action required |

### 2.2 Duplicates

- Total duplicate rows identified: TBD
- Rows after deduplication: TBD
- Assessment of duplication patterns: TBD

### 2.3 Anomalies

- Sessions with > 1 event per second (bot activity): TBD
- Sessions with > 100 distinct products (anomalous behaviour): TBD
- Sessions with fewer than 3 events (insufficient for modelling): TBD
- Outlier session durations: TBD

---

## 3. Event Distribution

*To be completed in iteration 1*

| Event Type | Count | Percentage |
|---|---|---|
| Product view | TBD | TBD |
| Add to cart | TBD | TBD |
| Purchase | TBD | TBD |

**Key observation:** Purchase events are expected to represent a small fraction
of all events, confirming class imbalance as a central modelling challenge.

---

## 4. Session-Level Statistics

*To be completed in iteration 1*

| Statistic | Value |
|---|---|
| Total sessions | TBD |
| Sessions with purchase | TBD |
| Sessions without purchase | TBD |
| Purchase session rate (%) | TBD |
| Mean events per session | TBD |
| Median events per session | TBD |
| Mean session duration (seconds) | TBD |
| Median session duration (seconds) | TBD |
| Mean cart additions per session | TBD |
| Mean views per session | TBD |

---

## 5. User-Level Statistics

*To be completed in iteration 1*

| Statistic | Value |
|---|---|
| Total unique users | TBD |
| Users with at least one purchase (%) | TBD |
| Mean sessions per user | TBD |
| Mean purchases per user | TBD |
| Mean products viewed per user | TBD |
| Mean distinct brands per user | TBD |

---

## 6. Class Imbalance Assessment

*To be completed in iteration 1*

- Proportion of sessions with purchase: TBD
- Proportion of sessions without purchase: TBD
- Imbalance ratio: TBD
- **Implication for modelling:** Class imbalance will be addressed through
  balanced subsampling for non-LightGBM models and class weight adjustment
  for LightGBM. Evaluation will prioritise AUC-PR and F1-score over raw
  accuracy.

---

## 7. Feature Correlation Analysis

*To be completed in iteration 1*

Key correlations with the target variable `has_purchase` to be reported here,
based on session-level aggregated features. Expected findings based on the
reference paper (Tokuç & Dağ, 2025):

- `cart_count` — expected moderate positive correlation with purchase
- `purchase_count` — expected strong positive correlation (near-tautological)
- `event_per_product` — expected negative correlation (indecision signal)
- `session_time_secs` — expected weak positive correlation
- `view_count` — expected weak negative correlation (browsing without intent)

Actual correlations to be documented here after EDA is completed.

---

## 8. Temporal Patterns

*To be completed in iteration 1*

- Purchase rate by day of week: TBD
- Purchase rate by hour of day: TBD
- Volume trends over the data collection period: TBD
- Seasonal or campaign-driven spikes: TBD

**Implication:** Temporal features are expected to carry predictive signal
and will be included in feature engineering.

---

## 9. Product and Category Insights

*To be completed in iteration 1*

- Number of unique products: TBD
- Number of unique categories: TBD
- Number of unique brands: TBD
- Distribution of purchase rate by category: TBD
- Price distribution across sessions: TBD

---

## 10. Key Findings and Implications for Modelling

*To be completed in iteration 1*

This section will summarise the most important findings from the EDA and state
their direct implications for data preparation and modelling decisions. At
minimum it will address:

- Confirmation or revision of the class imbalance handling strategy
- Any data quality issues that require additional preprocessing steps beyond
  those planned in the business understanding document
- Any unexpected feature distributions or correlations that change the
  feature engineering plan
- Any temporal patterns that affect the train-test split strategy
- Any findings that require revision of the business understanding document

---

## 11. Notebook Reference

All analysis supporting this document is in:

- `notebooks/01_data_understanding.ipynb`

---

## 12. Reference

Tokuç, A. A. & Dağ, T. (2025). Predicting User Purchases From Clickstream
Data: A Comparative Analysis of Clickstream Data Representations and Machine
Learning Models. *IEEE Access*, 13, 43796–43817.
https://doi.org/10.1109/ACCESS.2025.3548267

---

*This document is completed during iteration 1 and updated if data findings
in later iterations require revision. See git commit history for version
changes.*
