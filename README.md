# AI Loan Approval Decision Support System

An end-to-end Machine Learning and Business Intelligence solution designed to assist financial institutions in predicting loan defaults and making data-driven credit risk decisions. 

This project aims to predict whether a loan applicant is likely to default using historical Lending Club data, enabling risk management teams to approve, deny, or adjust loan terms.

---

## 📂 Project Structure

```text
Loan-Risk-Decision-System/
├── data/
│   ├── raw/                  # Contains the original 1.67GB Lending Club CSV (ignored by Git)
│   └── processed/            # Cleaned, optimized, and encoded CSV for ML (ignored by Git)
├── notebooks/
│   └── Data_Preprocessing.ipynb  # Interactive Jupyter Notebook for raw data ingestion and cleaning
├── models/                   # Folder for saving trained ML models (Phase 2)
├── powerbi/                  # Folder for Power BI dashboard files (Phase 3)
├── reports/                  # Data science and business reports
├── requirements.txt          # Python dependencies
├── .gitignore                # Specifies ignored files to keep repo clean
└── README.md                 # Project overview and documentation
```

---

## ⚙️ Setup & Installation

### 1. Prerequisites
- **Python**: Version `3.10` to `3.13`
- **Pip**: Latest version recommended

### 2. Installation
Clone this repository and install the dependencies from the project root:
```bash
pip install -r requirements.txt
```

### 3. Placing Raw Data
Ensure the Lending Club accepted loans CSV file `accepted_2007_to_2018Q4.csv` is placed inside `data/raw/`. 

*(Note: The CSV file is approximately 1.67 GB and is ignored by Git in `.gitignore` to avoid large file commits).*

---

## 📊 Phase 1: Data Preprocessing Details

In Phase 1, we focus on transforming the massive raw Lending Club dataset into a clean, model-ready format while optimizing memory consumption.

### 1. Selected Features
The following features are loaded and processed:
- **Demographics & Financials**: `annual_inc`, `emp_length`, `home_ownership`, `addr_state`
- **Loan Details**: `loan_amnt`, `term`, `int_rate`, `installment`, `purpose`
- **Credit Profile**: `dti`, `delinq_2yrs`, `fico_range_low`, `fico_range_high`, `inq_last_6mths`, `open_acc`, `pub_rec`, `revol_bal`, `revol_util`, `total_acc`, `mort_acc`
- **Loan Status (Target)**: `loan_status` (Binary: Good vs. Bad)
- **Metadata**: `grade`, `sub_grade`, `application_type`, `verification_status`

### 2. Preprocessing & Engineering Workflow
* **Memory-Efficient Ingestion**: Reads the CSV line-by-line using a random skip parser, loading exactly 250,000 random records using `random_state = 42`.
* **Target Definition**: 
  - `0 (Good Loan)`: Fully Paid
  - `1 (Bad Loan)`: Charged Off, Default
  - All other intermediate categories (e.g. Current, Late, Grace Period) are excluded to ensure a clean binary classification setting.
* **Missing Value Imputation**:
  - `emp_length` mapped to ordinal values; NaNs encoded as `-1` ("Unknown").
  - `mort_acc` imputed using the median count based on `total_acc` grouping (account-level correlation).
  - `revol_util` and `dti` imputed with their respective medians.
* **Multicollinearity Prevention**: `grade` is removed from the feature set in favor of `sub_grade` which is more granular and avoids perfect collinearity.
* **Feature Encoding**:
  - **Binary**: `term` (36m -> 0, 60m -> 1), `application_type` (Individual -> 0, Joint -> 1)
  - **Ordinal**: `sub_grade` (A1 -> 0, A2 -> 1, ..., G5 -> 34), `emp_length` (-1 to 10)
  - **One-Hot**: `home_ownership`, `verification_status`, `purpose`, `addr_state`
* **Data Downcasting**: Numeric types are optimized (e.g., float64 downcasted to float32, int64 downcasted to int16/int32) to reduce memory footprints from ~1.5 GB to ~25 MB.

---

## 📈 Execution of Phase 1

To run the data preprocessing notebook, start Jupyter:
```bash
jupyter notebook notebooks/Data_Preprocessing.ipynb
```
Select "Run All" from the Jupyter interface. The notebook is fully self-contained and will generate the cleaned dataset at `data/processed/cleaned_loans.csv`.
