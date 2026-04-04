# Purchase Intent Prediction for Inventory Management

## Overview

This project builds a purchase intent prediction model from e-commerce clickstream data and uses the model's output as a forward-looking demand signal to improve inventory management decisions. The core idea is that clickstream data captures user intent before it converts to a purchase, providing an earlier and richer demand signal than sales history alone.

Traditional inventory management relies on historical sales data, which only reflects demand after transactions have completed. This creates a structural lag — the system cannot see latent demand or demand that is about to materialise. By predicting purchase likelihood at the session level and aggregating those predictions to the product level, we generate demand signals that are more timely than sales-history-based forecasting.

---

## Research Foundation

This project replicates and extends the classification framework from:

> Tokuç, A. A. & Dağ, T. (2025). Predicting User Purchases From Clickstream Data: A Comparative Analysis of Clickstream Data Representations and Machine Learning Models. *IEEE Access*, 13, 43796–43817. https://doi.org/10.1109/ACCESS.2025.3548267

The paper demonstrates that a hybrid data representation — combining aggregated session-level features with recent sequential user actions — paired with LightGBM achieves superior purchase prediction performance (AUC-ROC: 0.9865) on real-world e-commerce clickstream data. This project uses that classification framework as the foundation for a downstream inventory management application that the original paper does not attempt.

---

## Dataset

**eCommerce Behavior Data from a Multi-Category Store**
- Source: [Kaggle — REES46](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store)
- Coverage: October 2019 to April 2020
- Scale: 233+ million raw events
- Event types: product view, add to cart, purchase
- Schema: `event_time`, `event_type`, `product_id`, `category_id`, `category_code`, `brand`, `price`, `user_id`, `user_session`

---

## Project Structure
```
├── data/
│   ├── raw/                        # Raw downloaded dataset (not committed to repo)
│   └── processed/                  # Cleaned and engineered datasets
├── docs/
│   ├── business_understanding.md   # CRISP-DM Phase 1
│   ├── data_understanding.md       # CRISP-DM Phase 2
│   ├── data_preparation.md         # CRISP-DM Phase 3
│   ├── modelling.md                # CRISP-DM Phase 4
│   ├── evaluation.md               # CRISP-DM Phase 5
│   └── deployment.md               # CRISP-DM Phase 6
├── notebooks/
│   ├── 01_data_understanding.ipynb
│   ├── 02_data_preparation.ipynb
│   ├── 03_modelling.ipynb
│   ├── 04_evaluation.ipynb
│   └── 05_demand_signal.ipynb
├── src/
│   ├── data/
│   │   ├── preprocessing.py
│   │   └── feature_engineering.py
│   ├── models/
│   │   ├── baseline.py
│   │   ├── train.py
│   │   └── evaluate.py
│   └── demand/
│       └── signal.py               # Product-level demand signal aggregation
├── tests/
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Methodology

The project follows the CRISP-DM framework and has two sequential technical components.

**Component 1 — Purchase prediction model**

A binary classification model predicting whether a session results in a purchase. Modelling follows a progression from simple to complex:

1. Naive baseline — majority class predictor
2. Logistic regression
3. Decision tree
4. Random forest
5. Gradient boosting
6. LightGBM — final benchmark consistent with Tokuç & Dağ (2025)

Three data representations are evaluated for each model, following the paper:
- Aggregated session attributes
- Flattened last N actions per session
- Hybrid — combination of both

**Component 2 — Demand signal construction**

Session-level predicted purchase probabilities are aggregated to the product level over rolling time windows to generate a forward-looking demand signal. This signal is compared against a sales-history baseline using a temporal holdout simulation, which also serves as a simulated A/B test to estimate business impact.

---

## Evaluation

**Technical metrics**

| Metric | Purpose |
|---|---|
| AUC-ROC | Primary discrimination metric |
| AUC-PR | Performance under class imbalance |
| F1-Score (purchase class) | Balance of precision and recall |
| Geometric Mean | Imbalance-aware classification quality |
| IBA | Imbalance-aware overall accuracy |
| Log Loss | Model calibration quality |

**Business KPIs**

| KPI | Description |
|---|---|
| Stockout rate | Frequency of running out of stock on high-intent products |
| Overstock rate | Excess inventory on low-intent products |
| Forecast error (MAPE) | Intent-augmented forecast vs sales-history baseline |
| Forecast lead time | Days earlier the intent signal predicts a demand spike |
| Lost sales reduction | Revenue recovered by avoiding stockouts |

---

## Getting Started

### Prerequisites

- Python 3.10+
- Apache Spark 3.3.2 (for large-scale data processing)
- See `requirements.txt` for full dependencies

### Installation
```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
pip install -r requirements.txt
```

### Data

Download the dataset from [Kaggle](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store) and place the files in `data/raw/`. The raw data files are excluded from version control via `.gitignore`.

---

## Documentation

Full phase-by-phase documentation is maintained in the `docs/` folder following the CRISP-DM framework. Start with [`docs/business_understanding.md`](docs/business_understanding.md).

---

## License

This project is for research and educational purposes. The dataset is subject to the terms and conditions of the REES46 data provider.
