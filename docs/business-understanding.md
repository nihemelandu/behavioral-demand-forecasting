# Business Understanding

**CRISP-DM Phase 1**  
**Iteration 0 Deliverable**  
*Document version 2.0*

---

## 1. Strategic Goal

To use session-level purchase intent signals derived from e-commerce clickstream
data to predict conversion likelihood, simulate the impact of targeted marketing
interventions under uncertainty, and provide actionable insights that improve
expected revenue and marketing efficiency.

---

## 2. Business Problem Statement

E-commerce platforms generate large volumes of clickstream data capturing how
users interact with products before making a purchase decision. Most of this
signal is unused for real-time decision-making. The core business problem has
three dimensions:

- Not all sessions are equally likely to convert. Treating all sessions
  identically wastes marketing resources on sessions that would have converted
  anyway and misses sessions that could have been nudged to convert with the
  right intervention.
- The effect of potential interventions — discounts, product recommendations,
  urgency messaging — is unknown for any given session. Historical data does
  not record what interventions were shown, making it impossible to estimate
  true causal effects from observation alone.
- Decision-makers need to act under uncertainty. A point estimate of expected
  lift is not sufficient — the distribution of possible outcomes, including
  downside risk, is what drives sound resource allocation.

This project addresses all three dimensions through a pipeline that moves from
probabilistic conversion prediction, through counterfactual simulation of
hypothetical interventions, to scenario-based analysis of expected outcomes
under uncertainty.

---

## 3. Technical Goal

To build an end-to-end probabilistic modelling and simulation pipeline that:

1. Predicts session-level conversion probability with quantified uncertainty
   using e-commerce clickstream behavioural features
2. Simulates the impact of hypothetical marketing interventions on conversion
   through counterfactual feature modification — not a randomised experiment —
   under explicitly stated modelling assumptions
3. Quantifies outcome uncertainty across intervention scenarios using Monte
   Carlo sampling
4. Compares interventions by expected conversion lift, revenue impact, and
   downside risk to support marketing prioritisation decisions

---

## 4. Stakeholders

| Stakeholder | Interest | What they need from this project |
|---|---|---|
| Marketing team | Intervention prioritisation | Which intervention to deploy, for which sessions, with what expected lift |
| Commercial / pricing team | Revenue impact | Expected incremental revenue per intervention with confidence intervals |
| Finance | Budget justification | Distribution of revenue outcomes including downside risk |
| Data / analytics team | Model validity and maintenance | Documented methodology, reproducible code, feature importance analysis |

---

## 5. Business KPIs

These are the metrics that determine whether the project delivers measurable
value to the business, independent of how the model performs technically.

**Conversion efficiency**

| KPI | Definition |
|---|---|
| Expected conversion lift | Mean increase in conversion rate under each simulated intervention relative to the no-intervention baseline |
| Intervention reliability | Probability that the simulated lift exceeds a minimum meaningful threshold |
| Session targeting precision | Proportion of sessions correctly identified as high-intent and worth intervening on |

**Revenue impact**

| KPI | Definition |
|---|---|
| Expected incremental revenue | Mean additional revenue estimated across simulated intervention scenarios |
| Revenue impact confidence interval | Range of expected incremental revenue at the 90% confidence level from Monte Carlo simulation |
| Risk-adjusted intervention rank | Ranking of interventions by expected lift adjusted for probability of underperformance |

---

## 6. Technical Metrics

Model-level metrics used to evaluate classification and probabilistic prediction
performance.

| Metric | Purpose |
|---|---|
| AUC-ROC | Primary discrimination metric |
| AUC-PR | Performance under class imbalance |
| F1-Score (purchase class) | Balance of precision and recall for the minority class |
| Geometric Mean | Imbalance-aware overall classification quality |
| IBA (Index Balanced Accuracy) | Imbalance-aware accuracy metric |
| Log Loss | Model calibration quality |
| Brier Score | Accuracy of probabilistic predictions |

---

## 7. Technical Plan

### Step 1: Data Preparation

**Dataset:** eCommerce Behavior Data from a Multi-Category Store (REES46 /
Kaggle). Covers October 2019 to April 2020. Event types: product view,
add-to-cart, purchase.

**Key data constraint:** The dataset records only three event types. Dwell
time is approximated from timestamp differences between consecutive events.
Traffic source and search queries are not available. No interventions shown
to users were recorded.

**Features engineered:**

*Behavioural:*
- Product view sequences and view counts
- Add-to-cart events and cart-to-view ratio
- Session duration (derived from first and last event timestamps)
- Time since last event within session
- Number of distinct products, categories, and brands viewed
- Event frequency (events per unit time)
- User type (first-time vs returning)

*Product-level:*
- Product price
- Category and brand encoding
- Price relative to session average

*Temporal:*
- Day of week and hour of day
- Is weekend indicator

*Target variable:*
- `label` — binary indicator: did this session result in a purchase?

**Preprocessing steps:**
- Deduplication
- Bot and anomaly removal (sessions with > 1 event per second or > 100
  distinct products)
- Post-purchase event trimming to prevent data leakage
- Removal of sessions with fewer than 3 events
- Missing value handling: `brand` encoded as "empty"; `category_code` excluded
- 90/10 stratified train-test split

---

### Step 2: Probabilistic Conversion Modelling

**Goal:** Predict P(conversion) per session with quantified uncertainty.

**Model progression (simple to complex):**

1. Naive baseline — majority class predictor
2. Logistic regression
3. Decision tree
4. Random forest
5. Gradient boosting
6. LightGBM — final benchmark consistent with Tokuç & Dağ (2025)

**Data representations evaluated:**
- Aggregated session attributes
- Flattened last N actions (N = 1, 5, 10)
- Hybrid — concatenation of aggregated and flattened

**Uncertainty quantification:** Post-hoc calibration via Platt scaling or
isotonic regression. LightGBM and gradient boosting are not natively Bayesian —
uncertainty is quantified through probability calibration, not posterior
inference. Brier score and log loss evaluate calibration quality.

**Feature selection:** Particle Swarm Optimisation (PSO) with AUC-ROC as
fitness function.

**Hyperparameter optimisation:** Tree-structured Parzen Estimator (TPE) via
hyperopt, with negative AUC-ROC as the objective function.

---

### Step 3: Counterfactual Simulation of Interventions

**What we are trying to achieve:** Estimate what the conversion outcome would
have been for each session if a specific marketing intervention had been
applied, when no such intervention was recorded in the historical data.

**Why:** The REES46 dataset contains no record of interventions shown to users.
There is no observed treatment variation. Causal inference from observational
data requires observed treatment and control groups, which do not exist here.
Counterfactual simulation is the methodologically appropriate response to this
constraint.

**Critical assumption:** Results are model-dependent estimates under stated
assumptions, not causally identified treatment effects. This is stated
explicitly in all reporting.

**Three hypothetical interventions:**

*Discount:* Reduce price-related features by a defined percentage to simulate
a price reduction.

*Recommendation:* Add a synthetic product view event for a related product
category to sessions that did not already contain one, simulating a relevant
product recommendation.

*Urgency:* Modify temporal features to reflect a compressed decision window,
simulating a time-limited offer.

**Method:** For each session, create a factual version (features as observed)
and a counterfactual version (features as modified under the intervention).
Pass both through the trained model to obtain factual and counterfactual
P(conversion). The difference is the estimated Individual Treatment Effect
(ITE) — model-estimated, not experimentally identified.

---

### Step 4: Monte Carlo Simulation

**What we are trying to achieve:** A full distribution of expected conversion
outcomes and revenue impact per intervention, not just a point estimate.

**Why:** Conversion is stochastic. A decision-maker who only sees expected
lift cannot assess downside risk. The full distribution — mean, confidence
intervals, tail risk — supports sound resource allocation under uncertainty.

**Method:**
- For each session and each intervention: treat counterfactual P(conversion)
  as the parameter of a Bernoulli distribution
- Draw a simulated conversion outcome (1 or 0)
- Repeat K = 10,000 times
- Within each repetition: sum conversions across all sessions → expected
  conversion count; multiply by average order value → expected revenue
- Extract from resulting distribution: mean, median, standard deviation,
  10th–90th percentile confidence intervals, probability of underperformance

---

### Step 5: Scenario-Based Analysis

**What we are trying to achieve:** A ranked, actionable comparison of
interventions expressed in business terms — expected lift, uncertainty, and
risk — that tells a stakeholder which intervention to prioritise and for
which sessions.

**Why:** The Monte Carlo simulation produces distributions for each
intervention in isolation. Scenario analysis synthesises those into a
comparative decision framework. Without this step the output is analytically
complete but not operationally actionable.

**Method:** Compare interventions on:

| Dimension | Metric |
|---|---|
| Expected lift | Mean of Monte Carlo distribution |
| Uncertainty | Standard deviation and IQR |
| Risk | P(lift < minimum meaningful threshold) |

Segment analysis by session characteristics to identify targeting rules —
which session types respond most strongly to each intervention.

---

### Step 6: Visualisation and Reporting

**Critical outputs and stakeholder relevance:**

| Visualisation | Purpose | Primary audience |
|---|---|---|
| Session-level conversion probability distribution | Validates model discrimination; shows baseline conversion landscape | Analytics team |
| Feature importance analysis | Identifies strongest behavioural predictors of conversion | Marketing and analytics teams |
| Intervention lift distributions with confidence intervals | Assesses intervention reliability across scenarios | Marketing and commercial teams |
| Revenue impact distributions | Expresses outcomes in monetary terms for budget decisions | Finance and commercial leadership |
| Intervention comparison summary | Ranked decision-support output readable by non-technical stakeholders | All stakeholders |
| Session segmentation heatmap | Identifies which session types respond to which interventions | Marketing team |

**Tooling:** matplotlib and seaborn for static analytical visualisations;
plotly for interactive distributions and scenario comparisons.

---

## 8. Project Scope

**In scope**
- Binary classification model predicting session-level conversion probability
- Probabilistic uncertainty quantification through model calibration
- Counterfactual simulation of three hypothetical interventions: discount,
  recommendation, urgency messaging
- Monte Carlo simulation of expected conversion and revenue outcomes
- Scenario-based analysis ranking interventions by expected lift, uncertainty,
  and risk
- Session segmentation to identify intervention targeting rules
- Reproducible Python workflows with version control
- Visualisation of behavioural patterns, simulated outcomes, and uncertainty

**Out of scope**
- Real-time deployment in a live e-commerce environment
- True causal inference from randomised controlled experiment data
- Integration with a live marketing or CRM platform

**Key methodological constraint:** The REES46 dataset contains no record of
interventions shown to users. Counterfactual simulation estimates are
model-dependent and are not causally identified treatment effects. All
reporting states this assumption explicitly.

**Key data constraint:** The dataset records only three event types: product
view, add-to-cart, and purchase. Dwell time is approximated from timestamp
differences. Traffic source and search queries are not available.

---

## 9. Iteration Plan

| Iteration | CRISP-DM Phase(s) | Deliverable | Key Question |
|---|---|---|---|
| 0 (Baseline) | Phase 1 | `docs/business_understanding.md` | Why are we building this? |
| 1 | Phase 2 | `docs/data_understanding.md` | What does the data tell us? |
| 2 | Phases 3–6 skeletal | End-to-end pipeline v1 | Can we run the full pipeline? |
| 3 | Phases 3–6 substantive | End-to-end pipeline v2 | Does the pipeline produce meaningful results? |
| 4 | Phases 3–6 refined | TBD from iteration 3 findings | To be scoped after iteration 3 |

---

## 10. Success Criteria

The project will be considered successful if:

- The classification model achieves AUC-ROC ≥ 0.95 on the test set
- Predicted probabilities are well-calibrated as measured by Brier score
  and reliability diagrams
- Counterfactual simulation produces interpretable and directionally
  plausible intervention lift estimates for at least two of the three
  intervention types
- Monte Carlo simulation produces stable outcome distributions verified
  by convergence testing
- Scenario analysis produces a ranked intervention comparison with
  confidence intervals interpretable by a non-technical stakeholder
- All workflows are reproducible from a clean environment using documented
  dependencies and version-controlled code

---

## 11. References

**Primary reference**

Tokuç, A. A. & Dağ, T. (2025). Predicting User Purchases From Clickstream
Data: A Comparative Analysis of Clickstream Data Representations and Machine
Learning Models. *IEEE Access*, 13, 43796–43817.
https://doi.org/10.1109/ACCESS.2025.3548267

**Supporting references**

Huang, T. & Van Mieghem, J. A. (2014). Clickstream Data and Inventory
Management: Model and Empirical Analysis. *Production and Operations
Management*, 23(3), 333–347.
https://doi.org/10.1111/poms.12046

Martínez-de-Albéniz, V., Planas, A. & Nasini, S. (2020). Using Clickstream
Data to Improve Flash Sales Effectiveness. *Production and Operations
Management*, 29(11), 2508–2531.
https://doi.org/10.1111/poms.13238

Jiang, Z., Li, J. & Zhang, D. (2024). A High-Dimensional Choice Model for
Online Retailing. *Management Science*, 71(4), 3320–3339.
https://doi.org/10.1287/mnsc.2020.02715

---

*This document is updated as the project progresses through CRISP-DM phases.
See git commit history for version changes.*
