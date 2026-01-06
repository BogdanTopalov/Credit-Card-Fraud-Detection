# Credit-Card-Fraud-Detection - End-to-End Microsoft Fabric Pipeline

This is an end-to-end fraud detection data pipeline built entirely in Microsoft Fabric, following a Lakehouse (Medallion) architecture and real-world fraud detection requirements.

The goal was not just to train a machine learning model, but to design a production-style data pipeline that:
- ingests raw transaction data
- cleans and standardizes it
- engineers meaningful fraud-related features
- trains an explainable ML model
- applies recall-first decision logic
- produces analytics-ready outputs for reporting and monitoring

The solution is aligned with enterprise data engineering practices.


### Business Problem

A credit card company wants to identify fraudulent transactions while minimizing missed fraud.

Key requirement:
- It is acceptable to flag legitimate transactions as suspicious
- Missing fraudulent transactions is considered more costly

This leads to a recall-first optimization strategy, where the model is tuned to catch as much fraud as possible, even at the cost of higher false positives.

### Architecture

The solution follows a Bronze → Silver → Gold Lakehouse architecture.

### Data Flow

```
Raw CSV Files
  ↓
Bronze Layer (Raw Transactions)
  ↓
Silver Layer (Cleaned & Typed Data)
  ↓
Gold Layer (ML Features)
  ↓
Model Training (Train only)
  ↓
Batch Scoring (Test / New Data)
  ↓
Gold Fraud Predictions (Analytics-ready)
```

### Fabric Components Used
- Microsoft Fabric Lakehouse (Delta tables)
- Fabric Notebooks (PySpark)
- Fabric Data Pipelines (Notebook orchestration)
- ML model persistence
- Gold tables for BI and reporting

### Dataset

https://www.kaggle.com/datasets/kartik2112/fraud-detection

The dataset represents anonymized credit card transactions and includes:
- transaction metadata (time, amount, location)
- customer attributes (age, city population)
- merchant attributes (category, location)
- a binary fraud label (is_fraud)

The data is highly imbalanced, which is typical for fraud detection problems.

### Project Structure

```
fraud-detection-fabric/
│
├── notebooks/
│   ├── 01_ingest_bronze.ipynb
│   ├── 02_clean_silver.ipynb
│   ├── 03_feature_engineering.ipynb
│   ├── 04_train_model.ipynb
│   └── 05_scoring.ipynb
│
└── README.md
```

## Data Layers

### Bronze Layer
- Raw transaction files ingested into Fabric Lakehouse
- No transformations applied
- Acts as immutable source of truth

### Silver Layer
- Cleaned and standardized data
- Proper data types (timestamps, numerics)
- Duplicates and invalid records removed
- Train and test datasets kept separate to avoid data leakage

### Gold Layer
- ML-ready feature tables
- Personally identifiable information removed from features
- Transaction identifiers preserved for traceability
- Separate tables for:
    - training features
    - test features
    - scored predictions

## Feature Engineering

Features are designed to capture fraud-relevant behavior, not personal identity.

Key feature groups:
- Time-based: transaction hour, day of week, month
- Amount-based: raw amount and logarithmic transformation
- Geographic: distance between customer and merchant
- Customer context: age, city population
- Merchant category: encoded using train-only category mapping

Categorical encoding is learned only from training data and applied to test data to prevent data leakage.

## Modeling Strategy

### Model Choice
- Logistic Regression
- Chosen for:
    - interpretability
    - probability outputs
    - explicit threshold control
    - common use in financial fraud systems

### Evaluation Approach
- Model is trained only on training data
- Evaluation is performed only on test data
- Accuracy is not the primary metric

### Recall-First Optimization

Instead of using the default 0.5 classification threshold:
- a lower probability threshold is applied
- recall is intentionally increased
- false positives are accepted as a business tradeoff

This reflects real-world fraud prevention logic.

## Scoring & Predictions

The trained model is saved and reused for batch scoring.

The final predictions table includes:
- transaction identifiers
- transaction context (time, amount, category)
- fraud probability
- suspicious transaction flag
- true fraud label (for evaluation)

This table is analytics-ready and designed for:
- Power BI dashboards
- operational review
- fraud monitoring workflows

## Key Design Decisions
- Separation of features and identifiers
- No data leakage between train and test
- Simple, explainable model over complex black-box models
- Business-driven metric selection (recall over accuracy)
- Production-style overwrite and schema control in Delta tables

## Skills Demonstrated
- Microsoft Fabric Lakehouse architecture
- Medallion (Bronze/Silver/Gold) design
- PySpark data transformations
- Feature engineering for ML
- Fraud detection modeling strategy
- Threshold tuning based on business constraints
- Batch scoring pipelines
- Analytics-ready data modeling

## Future Improvements
- Real-time or near-real-time ingestion
- Cost-based threshold optimization
- Model versioning and monitoring
- Alerting and automated actions
- Power BI executive dashboard
