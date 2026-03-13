# Cost-Sensitive Fraud Detection with Cascade Decision Framework

> A decision-centric approach to CNP (Card-Not-Present) fraud detection that reduces total fraud cost by **36.1%** compared to the standard F1-Score baseline.

## Why This Matters

Most fraud detection systems optimize for statistical metrics (F1, AUC). But a $5,000 undetected fraud causes far more damage than a $30 false block. When error costs are asymmetric, the decision threshold should be too.

This project implements a **cascade decision framework** that:

1. Replaces the single binary threshold with **three decision zones** optimized by a cost function
2. Adds a **cost-aware review queue** that prioritizes manual reviews by expected financial impact
3. Delivers **measurable ROI in monetary units**, not just metric improvements

## Key Results

| Metric | F1 Baseline | Cost-Optimized | Cascade |
|---|---|---|---|
| Total Cost | $745,534 | $570,975 | **$476,453** |
| Recall | 46.6% | 70.0% | ~85% |
| Cost Reduction | -- | 23.4% | **36.1%** |
| Savings | -- | $174,559 | **$269,082** |

**Cascade zone allocation (test set):**

| Zone | Threshold | Volume | Action |
|---|---|---|---|
| Zone 1: Auto-approve | score < 0.038 | 88.9% | Instant approval |
| Zone 2: Manual review | 0.038 <= score < 0.322 | 9.0% | Prioritized by expected cost |
| Zone 3: Auto-block | score >= 0.322 | 2.1% | Instant block |

## Architecture

```
Transaction --> Scoring Module --> Cost Optimizer --> Cascade Router
                (LightGBM +        (asymmetric       |
                 iForest)           cost function)    |-- Zone 1: auto-approve (88.9%)
                                                      |-- Zone 2: review queue (9.0%)
                                                      |      \-- Priority = P(fraud|x) * amount
                                                      |-- Zone 3: auto-block (2.1%)
```

## Project Structure

```
.
|-- notebooks/
|   |-- fraud_detection_cascade.ipynb   # Full analysis (EDA to production framework)
|-- results/
|   |-- figures/                        # Key visualizations
|-- requirements.txt
|-- LICENSE
|-- README.md
```

## Methodology

### 1. Data

**IEEE-CIS Fraud Detection Dataset** (Vesta Corporation via Kaggle)

- 590,540 real e-commerce transactions
- 6 months of data, 3.5% fraud rate (27:1 imbalance)
- 434 features (transaction + identity)
- Temporal train/test split (~4 months train, ~2 months test), no data leakage

### 2. Feature Engineering

- Removal of columns with > 60% missing values
- Selection of top V-features by mutual information with fraud label
- Temporal features: hour bins, day of week, weekend flag
- Interaction features: amount/card aggregations
- Target encoding for high-cardinality categoricals
- Anomaly score from Isolation Forest as meta-feature

### 3. Model Selection

Benchmark of 5 classifiers with temporal cross-validation:

| Model | AUC-ROC | Training Time |
|---|---|---|
| LightGBM | **0.9113** | 12s |
| XGBoost | 0.9087 | 45s |
| Random Forest | 0.8834 | 38s |
| Logistic Regression | 0.8456 | 3s |
| Neural Network (MLP) | 0.8721 | 28s |

LightGBM selected as primary scorer.

### 4. Cost-Sensitive Threshold Optimization

Instead of maximizing F1 (threshold = 0.220), we minimize total cost:

```
Total Cost = FN_count * avg_fraud_amount + FP_count * review_cost
```

Where:
- `avg_fraud_amount`: mean transaction value of actual frauds (dynamic)
- `review_cost`: $16 per false positive (customer friction + operational cost)

Optimal binary threshold: **0.045** (vs 0.220 for F1)

### 5. Cascade Decision Framework

Two thresholds optimized simultaneously to create three decision zones. Zone 2 includes a **cost-aware prioritization queue**:

```
Priority(x) = P(fraud | x) * TransactionAmount(x)
```

With a simulated capacity of 100 reviews/day at 95% analyst accuracy, the cascade achieves $103K additional savings vs naive score-ordering.

### 6. Sensitivity Analysis

The cascade maintains positive savings across all tested scenarios:
- FP cost range: $5 to $100
- FN cost multiplier: 0.5x to 3.0x
- Review capacity: 50 to 500/day

## Technical Stack

- **Language**: Python 3.10+
- **ML**: LightGBM, scikit-learn, XGBoost
- **Data**: pandas, NumPy
- **Visualization**: matplotlib, seaborn
- **Anomaly Detection**: Isolation Forest (scikit-learn)

## How to Run

```bash
# Clone the repository
git clone https://github.com/NorianePadilha/fraud-detection-cascade.git
cd fraud-detection-cascade

# Install dependencies
pip install -r requirements.txt

# Download the dataset from Kaggle
# https://www.kaggle.com/c/ieee-fraud-detection/data
# Place train_transaction.csv and train_identity.csv in data/

# Run the notebook
jupyter notebook notebooks/fraud_detection_cascade.ipynb
```

## Business Impact

For a payment processor handling 50,000 transactions/day:
- **Projected annual savings**: $23.7M
- **Break-even point**: 64 transactions/day
- **Review queue efficiency**: +$103K vs score-based ordering

## Academic Context

This analysis was developed as part of an MBA thesis at PUC-RS (Pontifical Catholic University of Rio Grande do Sul), focusing on the intersection of machine learning and business decision-making for fraud prevention.

**Thesis**: "FraudShield: Cost-Asymmetric Decision Cascade for CNP Fraud Detection"

The framework demonstrates that the gap between ML performance and business value lies not in the model, but in the decision layer that operates on top of it.

## Key Insight

> Optimizing a model is an ML problem. Optimizing the decision is a business problem. This project bridges both.

## License

MIT License. See [LICENSE](LICENSE) for details.


