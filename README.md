# 📉 Customer Churn Prediction Dashboard

An end-to-end machine learning project that predicts which customers are likely to churn and
surfaces those insights in an interactive Streamlit dashboard — built for business users to
explore churn drivers, evaluate model performance, and score customers in real time.

[![CI](https://github.com/<your-username>/churn-prediction-dashboard/actions/workflows/ci.yml/badge.svg)](https://github.com/<your-username>/churn-prediction-dashboard/actions/workflows/ci.yml)
![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Streamlit](https://img.shields.io/badge/built%20with-Streamlit-FF4B4B)

---

## 🔗 Live Demo

> Deploy this app for free on [Streamlit Community Cloud](https://streamlit.io/cloud) in about
> two minutes (see [Deployment](#-deployment)), then drop the live link here:
>
> **[https://your-app-name.streamlit.app](https://your-app-name.streamlit.app)**

---

## 📸 Preview

| Overview | Model Insights |
|---|---|
| ![Churn distribution](reports/figures/churn_distribution.png) | ![ROC curve](reports/figures/roc_curve.png) |

| Confusion Matrix | Feature Importance |
|---|---|
| ![Confusion matrix](reports/figures/confusion_matrix.png) | ![Feature importance](reports/figures/feature_importance.png) |

---

## 💡 Why this project

Customer churn is one of the most expensive problems in subscription businesses — acquiring a
new customer typically costs far more than retaining an existing one. This project simulates a
realistic retention workflow:

1. **Understand** who is churning and why (EDA + dashboard analytics)
2. **Predict** churn risk before it happens (trained ML classifier)
3. **Act** by scoring individual customers or entire CSV batches through a self-serve UI

## ✨ Features

- **Interactive 4-page dashboard** (Streamlit + Plotly)
  - 🏠 **Overview** — KPIs (churn rate, revenue at risk), churn by contract/internet service, tenure & charges distributions
  - 🔍 **Customer Explorer** — filterable customer table with live segment churn rates
  - 🤖 **Model Insights** — model leaderboard, ROC curve, confusion matrix, feature importance, full classification report
  - 🎯 **Predict Churn** — score a single customer via form inputs (with a live risk gauge), or upload a CSV for batch scoring with a downloadable results file
- **Reproducible ML pipeline** — clean separation between data cleaning, feature preprocessing, and model training (`src/`)
- **Model comparison** — Logistic Regression, Random Forest, and XGBoost trained and benchmarked automatically; the best model (by ROC-AUC) is selected and persisted
- **Automated tests** (`pytest`) covering data integrity, the preprocessing pipeline, and model sanity checks
- **CI pipeline** (GitHub Actions) that retrains the model and runs the full test suite on every push
- **EDA notebook** documenting the analysis that informed feature selection

## 🧰 Tech Stack

| Layer | Tools |
|---|---|
| Language | Python 3.10+ |
| Data & ML | pandas, NumPy, scikit-learn, XGBoost |
| Visualization | Matplotlib, Seaborn, Plotly |
| Dashboard | Streamlit |
| Testing / CI | pytest, GitHub Actions |
| Model persistence | joblib |

## 📊 Dataset

[IBM Telco Customer Churn](https://github.com/IBM/telco-customer-churn-on-icp4d) — 7,043 customers
across 21 attributes (demographics, account information, subscribed services) with a binary churn
label. Overall churn rate is ~26.5%, a realistic class imbalance handled via `class_weight="balanced"`
during training.

## 🏗️ Project Structure

```
churn-prediction-dashboard/
├── app/
│   └── dashboard.py            # Streamlit application (4-page dashboard)
├── src/
│   ├── config.py                # Paths & constants
│   ├── data_preprocessing.py    # Load & clean raw data
│   ├── pipeline.py              # sklearn ColumnTransformer (shared train/serve)
│   └── train_model.py           # Trains, compares & persists the best model
├── notebooks/
│   └── 01_exploratory_data_analysis.ipynb
├── tests/
│   ├── test_data_preprocessing.py
│   ├── test_pipeline.py
│   └── test_model.py
├── data/
│   ├── raw/telco_churn.csv      # Source dataset
│   └── processed/               # Cleaned data (generated)
├── models/                      # Trained model, preprocessor & metrics (generated)
├── reports/figures/             # Diagnostic plots (generated)
├── .github/workflows/ci.yml     # CI: retrain + test on every push
├── requirements.txt
└── README.md
```

## 🚀 Getting Started

### 1. Clone & install

```bash
git clone https://github.com/<your-username>/churn-prediction-dashboard.git
cd churn-prediction-dashboard
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Train the model

```bash
python -m src.train_model
```

This cleans the data, trains Logistic Regression / Random Forest / XGBoost, benchmarks them on a
held-out test set, and saves the winning model, preprocessor, metrics, and diagnostic plots.

### 3. Launch the dashboard

```bash
streamlit run app/dashboard.py
```

Then open **http://localhost:8501** in your browser.

### 4. Run the tests

```bash
pytest tests/ -v
```

## 📈 Model Performance

Three classifiers were trained and compared on a stratified 80/20 train/test split (metrics on
the held-out test set):

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| **Random Forest** ⭐ | 0.767 | 0.543 | 0.753 | 0.631 | **0.841** |
| Logistic Regression | 0.742 | 0.508 | 0.780 | 0.615 | 0.840 |
| XGBoost | 0.791 | 0.635 | 0.492 | 0.555 | 0.839 |

**Random Forest** was selected as the production model for its best ROC-AUC. Given the business cost
of *missing* a churner is typically higher than the cost of a false alarm (a retention offer sent to
a loyal customer is cheap; losing a customer is not), the model is tuned for **recall** via
`class_weight="balanced"` — catching ~75% of customers who actually churn.

> Re-run `python -m src.train_model` any time to regenerate these numbers — results are also
> written to `models/metrics.json` and viewable live in the dashboard's **Model Insights** tab.

## 🔑 Key Insights from EDA

- **Contract type is the strongest churn signal** — month-to-month customers churn far more than
  those on one- or two-year contracts.
- **Churn risk is concentrated in the first year** of the customer relationship.
- **Fiber optic customers without security/tech-support add-ons** are a high-risk segment —
  a clear target for proactive retention offers.
- **Higher monthly charges correlate with churn**, suggesting price sensitivity among newer,
  higher-spend customers.

Full analysis: [`notebooks/01_exploratory_data_analysis.ipynb`](notebooks/01_exploratory_data_analysis.ipynb)

## ☁️ Deployment

The dashboard deploys for free on [Streamlit Community Cloud](https://share.streamlit.io/):

1. Push this repo to your own GitHub account.
2. Go to [share.streamlit.io](https://share.streamlit.io/) → **New app**.
3. Point it at your repo, branch `main`, and file path `app/dashboard.py`.
4. Deploy — Streamlit Cloud installs `requirements.txt` automatically.

> Note: commit the `models/` directory (or add a build step that runs `python -m src.train_model`)
> so the deployed app has a trained model to load.

## 🗺️ Possible Extensions

- Hyperparameter tuning with `GridSearchCV` / `Optuna`
- SHAP values for per-customer explainability in the Predict tab
- A `/predict` REST API (FastAPI) alongside the dashboard for programmatic access
- Model monitoring / drift detection for production retraining triggers
- Docker Compose setup with a scheduled retraining job

## 📄 License

This project is licensed under the [MIT License](LICENSE).

## 🙋 About

Built as a portfolio project demonstrating an end-to-end applied ML workflow: data cleaning,
model selection, evaluation, and deployment behind a genuinely usable business-facing UI —
not just a notebook.

If you found this useful, consider ⭐ starring the repo!
