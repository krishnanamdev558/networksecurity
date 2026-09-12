# Network Security ML Pipeline: Phishing Detection System


![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

Short description of your project goes here...

An end-to-end machine learning platform for phishing detection that classifies websites as legitimate or malicious based on URL and domain characteristics. This production-grade system integrates data ingestion from MongoDB, automated ETL pipelines, multi-algorithm model training with hyperparameter optimization, and real-time inference capabilities via REST API.

---

## 📋 Table of Contents

- [Project Overview & Business Context](#project-overview--business-context)
- [Architecture & Data Flow](#architecture--data-flow)
- [ETL & Data Pipeline](#etl--data-pipeline)
- [Machine Learning & MLOps](#machine-learning--mlops)
- [Technical Stack](#technical-stack)
- [Repository Structure](#repository-structure)
- [Setup & Installation](#setup--installation)
- [Execution Guide](#execution-guide)
- [API Reference](#api-reference)
- [Configuration](#configuration)
- [Monitoring & Logging](#monitoring--logging)

---

## 🎯 Project Overview & Business Context

### Business Problem

Phishing attacks remain a critical cybersecurity threat, with attackers continuously evolving tactics to deceive users and compromise credentials. Manual identification of phishing websites is labor-intensive and error-prone. This project automates phishing detection through machine learning by analyzing URL and domain-level features to provide real-time classification.

### Objectives

- **Accuracy**: Build a high-precision classifier to minimize false negatives (missed phishing sites)
- **Scalability**: Design a pipeline capable of processing high-volume prediction requests
- **Production-Ready**: Implement end-to-end ML ops with experiment tracking, model versioning, and monitoring
- **Maintainability**: Structure code for reproducibility, modularity, and ease of deployment

### Solution Architecture

The system follows a classical supervised learning pipeline:
1. **Data Source**: Network security dataset with 30 engineered URL/domain features from MongoDB
2. **Target Variable**: Binary classification (0 = Legitimate, 1 = Phishing)
3. **Models Evaluated**: Random Forest, Decision Tree, Gradient Boosting, Logistic Regression, AdaBoost
4. **Deployment**: FastAPI-based microservice for real-time predictions
5. **Experiment Tracking**: MLflow + DagShub for reproducibility and model management

---

## 🏗️ Architecture & Data Flow

### End-to-End Workflow Diagram

```
┌────────────────────────────────────────────────────────────────────────────┐
│                          DATA INGESTION & PREPARATION                       │
│  ┌──────────────┐        ┌──────────────┐        ┌────────────────────┐   │
│  │  MongoDB     │───────▶│  Feature     │───────▶│  Train/Test Split  │   │
│  │  Collection  │        │  Store CSV   │        │  (80/20)           │   │
│  └──────────────┘        └──────────────┘        └────────────────────┘   │
└────────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                         DATA VALIDATION & QUALITY                          │
│  ┌──────────────────────┐      ┌──────────────────┐      ┌──────────────┐ │
│  │ Schema Validation    │──────▶│ Data Drift      │──────▶│ Valid/       │ │
│  │ (Column Existence)   │      │ Detection (KS   │      │ Invalid      │ │
│  │                      │      │ Test)           │      │ Segregation  │ │
│  └──────────────────────┘      └──────────────────┘      └──────────────┘ │
└────────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                        FEATURE ENGINEERING & TRANSFORMATION                 │
│  ┌──────────────────────┐      ┌──────────────────┐      ┌──────────────┐ │
│  │ Missing Value        │──────▶│ KNN Imputation  │──────▶│ Transformed  │ │
│  │ Detection (NaN)      │      │ (n_neighbors=3) │      │ NumPy Arrays │ │
│  └──────────────────────┘      └──────────────────┘      └──────────────┘ │
└────────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                     MODEL TRAINING & HYPERPARAMETER TUNING                  │
│  ┌─────────┐  ┌──────────┐  ┌────────────┐  ┌──────────────┐  ┌────────┐ │
│  │Random   │  │Decision  │  │ Gradient   │  │ Logistic     │  │AdaBoost│ │
│  │Forest   │  │Tree      │  │ Boosting   │  │ Regression   │  │        │ │
│  └─────────┘  └──────────┘  └────────────┘  └──────────────┘  └────────┘ │
│       │            │              │               │               │        │
│       └────────────┴──────────────┴───────────────┴───────────────┘        │
│                            │                                               │
│                            ▼                                               │
│              ┌──────────────────────────────┐                             │
│              │ Model Evaluation & Selection │                             │
│              │ (F1, Precision, Recall)      │                             │
│              └──────────────────────────────┘                             │
└────────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                    EXPERIMENT TRACKING & MODEL REGISTRY                     │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │ MLflow + DagShub: Log metrics, model artifacts, hyperparameters    │  │
│  │ Serialized Models: model.pkl (trained classifier)                  │  │
│  │ Preprocessor: preprocessor.pkl (KNN imputation pipeline)           │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                          INFERENCE & PREDICTION                            │
│  ┌──────────────┐      ┌──────────────────┐      ┌─────────────────────┐ │
│  │ REST API     │──────▶│ Preprocessing    │──────▶│ Model Inference &   │ │
│  │ (FastAPI)    │      │ (Transform Input)│      │ Output Generation   │ │
│  └──────────────┘      └──────────────────┘      └─────────────────────┘ │
│  • POST /predict                                   • CSV Export           │
│  • GET /train                                      • JSON Response        │
│  • File Upload (CSV)                                                       │
└────────────────────────────────────────────────────────────────────────────┘
```

### Component Interaction

```
┌─────────────────────────────────────────────────────────────────┐
│                     TrainingPipeline                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐                                           │
│  │ DataIngestion    │──── MongoDB ──── CSV ──── Split           │
│  └──────────────────┘                                           │
│           │                                                      │
│           ▼                                                      │
│  ┌──────────────────┐                                           │
│  │ DataValidation   │──── Schema ──── Drift Detection           │
│  └──────────────────┘                                           │
│           │                                                      │
│           ▼                                                      │
│  ┌──────────────────┐                                           │
│  │ DataTransformation│──── Imputation ──── Preprocessing        │
│  └──────────────────┘                                           │
│           │                                                      │
│           ▼                                                      │
│  ┌──────────────────┐                                           │
│  │ ModelTrainer     │──── Grid Search ──── Evaluation           │
│  └──────────────────┘──── MLflow Logging ──── Model Serialization│
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 ETL & Data Pipeline

### Data Extraction

**Source**: MongoDB Collection
- **Database**: `KRISHNA_AI`
- **Collection**: `NetworkData`
- **Connection**: SSL/TLS secured via environment variable `MONGODB_URL_KEY`
- **Data Format**: BSON documents (converted to Pandas DataFrame)

**Raw Data Handling**:
```python
# Loads from MongoDB collection
df = pd.DataFrame(list(collection.find()))
# Removes MongoDB metadata (_id)
df = df.drop(columns=["_id"])
# Replaces string 'na' with np.nan for consistent missing value representation
df.replace({"na": np.nan}, inplace=True)
```

**Data Batching**:
- Entire dataset loaded into memory (suitable for datasets < 10GB)
- Feature store exported to CSV: `data_ingestion/feature_store/phisingData.csv`

### Data Transformation

**Preprocessing Pipeline**:

| Step | Operation | Parameters | Output |
|------|-----------|-----------|--------|
| **Imputation** | KNN Missing Value Imputation | n_neighbors=3, weights='uniform' | Filled numerical arrays |
| **Feature Split** | Separate features from target | Target column: `Result` | X_train/X_test, y_train/y_test |
| **Label Encoding** | Convert target values | Replace -1 → 0 (binary) | Binary labels {0, 1} |
| **Array Serialization** | Save as NumPy binary | .npy format | Efficient storage/loading |

**Data Quality Checks**:

```yaml
Schema Validation:
  - Column count: Must match schema (31 columns)
  - Column names: Must align with data_schema/schema.yaml
  - Data types: All int64 (enforced)
  
Missing Values:
  - Detection: NaN identification
  - Imputation: KNN with k=3 neighbors
  - Strategy: Preserve statistical properties
  
Data Drift Detection (Kolmogorov-Smirnov Test):
  - Compares train/test distributions per feature
  - Threshold: p-value > 0.05 (no significant drift)
  - Reports: Saved to data_validation/drift_report/report.yaml
```

### Data Loading & Storage

**Output Destinations**:

| File | Location | Format | Purpose |
|------|----------|--------|---------|
| Feature Store | `data_ingestion/feature_store/` | CSV | Raw data snapshot |
| Training Set | `data_ingestion/ingested/train.csv` | CSV | Model training |
| Test Set | `data_ingestion/ingested/test.csv` | CSV | Evaluation |
| Transformed Train | `data_transformation/transformed/train.npy` | NumPy | Preprocessing input |
| Transformed Test | `data_transformation/transformed/test.npy` | NumPy | Preprocessing input |
| Preprocessor | `final_model/preprocessor.pkl` | Pickle | KNN imputation object |

**Directory Structure**:
```
Artifacts/
├── data_ingestion/
│   ├── feature_store/
│   │   └── phisingData.csv
│   └── ingested/
│       ├── train.csv
│       └── test.csv
├── data_validation/
│   ├── validated/
│   │   ├── train.csv
│   │   └── test.csv
│   └── drift_report/
│       └── report.yaml
├── data_transformation/
│   ├── transformed/
│   │   ├── train.npy
│   │   └── test.npy
│   └── transformed_object/
│       └── preprocessing.pkl
└── model_trainer/
    └── trained_model/
        └── model.pkl
```

---

## 🤖 Machine Learning & MLOps

### Model Architectures

**Candidate Models Evaluated**:

| Model | Type | Hyperparameters Tuned | Rationale |
|-------|------|----------------------|-----------|
| **Random Forest** | Ensemble (Bagging) | `n_estimators`: [8,16,32,128,256] | Robust to overfitting, handles non-linearity |
| **Decision Tree** | Tree-based | `criterion`: ['gini','entropy','log_loss'] | Interpretability, feature importance |
| **Gradient Boosting** | Ensemble (Boosting) | `learning_rate`: [.1,.01,.05,.001], `subsample`: [0.6-0.9], `n_estimators`: [8-256] | Powerful for complex patterns |
| **Logistic Regression** | Linear | None (baseline) | Fast baseline, probabilistic output |
| **AdaBoost** | Adaptive Boosting | `learning_rate`: [.1,.01,.001], `n_estimators`: [8-256] | Focus on misclassified samples |

### Hyperparameter Optimization

**Strategy**: Grid Search with Cross-Validation

```python
# Example: Gradient Boosting hyperparameter space
GridSearchCV(
    estimator=GradientBoostingClassifier(verbose=1),
    param_grid={
        'learning_rate': [0.1, 0.01, 0.05, 0.001],
        'subsample': [0.6, 0.7, 0.75, 0.85, 0.9],
        'n_estimators': [8, 16, 32, 64, 128, 256]
    },
    cv=5,  # 5-fold cross-validation
    scoring='f1'
)
```

### Model Evaluation Metrics

**Classification Metrics Tracked**:

| Metric | Formula | Interpretation | Target |
|--------|---------|-----------------|--------|
| **F1-Score** | $2 \times \frac{P \times R}{P + R}$ | Harmonic mean of precision & recall | Maximize (0-1) |
| **Precision** | $\frac{TP}{TP + FP}$ | % predicted phishing that are actual phishing | Minimize false positives |
| **Recall** | $\frac{TP}{TP + FN}$ | % actual phishing correctly identified | Minimize false negatives |
| **ROC-AUC** | Area under ROC curve | Discrimination across thresholds | Maximize (0-1) |

**Validation Strategy**:
- **Train-Test Split**: 80% training, 20% testing
- **Cross-Validation**: 5-fold CV during hyperparameter tuning
- **Threshold**: Minimum F1-score for production deployment: 0.6

### Experiment Tracking & Model Registry

**MLflow Integration**:
```
MLflow Setup:
├── Tracking Server: Remote (DagShub)
├── Experiment: "Network Security Classification"
├── Logged Artifacts per Run:
│   ├── Model object (scikit-learn format)
│   ├── Metrics: f1_score, precision, recall
│   ├── Parameters: hyperparameters used
│   └── Tags: model version, algorithm
└── Model Registry: Automatic versioning
```

**DagShub Integration**:
- **Repository Owner**: krishnanamdev558
- **Repository Name**: networksecurity
- **MLflow Backend**: DagShub remote storage
- **Purpose**: Centralized experiment tracking, reproducible runs

### Model Serialization & Storage

**Artifact Persistence**:
```python
# Final trained model
final_model/
├── model.pkl              # Best trained classifier
├── preprocessor.pkl       # KNN imputation pipeline
└── metrics.json          # Final evaluation metrics

# Best model selection criteria
selected_model = models[best_model_name]  
# where best_model_name = argmax(f1_score across all CV folds)
```

**Serialization Format**: Python pickle (.pkl)
- Preserves model state, hyperparameters, and learned parameters
- Used for inference: `load_object('final_model/model.pkl')`

---

## 🛠️ Technical Stack

### Data Engineering & Processing

| Tool | Version | Purpose |
|------|---------|---------|
| **Pandas** | Latest | DataFrame manipulation, CSV I/O, data cleaning |
| **NumPy** | Latest | Numerical arrays, missing value operations |
| **Scikit-Impute (KNNImputer)** | Latest | KNN-based missing value imputation |

### Machine Learning & Modeling

| Tool | Version | Purpose |
|------|---------|---------|
| **Scikit-Learn** | Latest | Classification models, metrics, preprocessing pipelines |
| | | • RandomForestClassifier |
| | | • DecisionTreeClassifier |
| | | • GradientBoostingClassifier |
| | | • LogisticRegression |
| | | • AdaBoostClassifier |

### MLOps & Experiment Tracking

| Tool | Version | Purpose |
|------|---------|---------|
| **MLflow** | Latest | Experiment logging, model versioning, artifact storage |
| **DagShub** | Latest | Remote MLflow backend, Git integration, experiment UI |

### Web Framework & Serving

| Tool | Version | Purpose |
|------|---------|---------|
| **FastAPI** | Latest | Async REST API framework with automatic docs |
| **Uvicorn** | Latest | ASGI server for FastAPI |
| **Python-Multipart** | Latest | File upload handling (multipart/form-data) |
| **Jinja2Templates** | Latest | HTML template rendering (prediction results) |

### Database & Data Storage

| Tool | Version | Purpose |
|------|---------|---------|
| **PyMongo** | Latest | MongoDB client with SSL/TLS support |
| **Certifi** | Latest | CA certificate handling for MongoDB connection |

### Environment & Configuration

| Tool | Version | Purpose |
|------|---------|---------|
| **Python-Dotenv** | Latest | Load environment variables from `.env` file |
| **PyYAML** | Latest | Schema and drift report configuration parsing |
| **Dill** | Latest | Advanced serialization for complex Python objects |

### Development & Testing

| Tool | Version | Purpose |
|------|---------|---------|
| **Pytest** | Optional | Unit testing framework |
| **Python** | 3.8+ | Runtime environment |

### Containerization (Optional)

| Tool | Version | Purpose |
|------|---------|---------|
| **Docker** | 20.10+ | Container orchestration |
| **Docker Compose** | 1.29+ | Multi-service orchestration |

---

## 📁 Repository Structure

```
networksecurity/
│
├── 📄 README.md                          # Project documentation (this file)
├── 📄 requirements.txt                   # Python package dependencies
├── 📄 setup.py                           # Package configuration & installation
├── 📄 main.py                            # Direct pipeline execution script
├── 📄 app.py                             # FastAPI web application
├── 📄 Dockerfile                         # Container image specification
├── 📄 push_data.py                       # MongoDB data ingestion utility
│
├── 📊 data_schema/                       # Data validation schemas
│   └── 📄 schema.yaml                    # Feature definitions and types
│
├── 📂 Network_Data/                      # Source data directory
│   └── 📄 phisingData.csv                # Raw phishing dataset (31 features)
│
├── 📂 networksecurity/                   # Main package source code
│   │
│   ├── 🔗 __init__.py                    # Package initialization
│   │
│   ├── ⚙️ constant/                      # Project constants & configurations
│   │   └── training_pipeline/
│   │       ├── 🔗 __init__.py
│   │       └── Constants: TARGET_COLUMN, ARTIFACT_DIR, SCHEMA_FILE_PATH, etc.
│   │
│   ├── 📋 entity/                        # Data class definitions
│   │   ├── 🔗 __init__.py
│   │   ├── config_entity.py              # Config: DataIngestionConfig, DataValidationConfig, etc.
│   │   └── artifact_entity.py            # Artifacts: DataIngestionArtifact, ModelTrainerArtifact, etc.
│   │
│   ├── 🔄 components/                    # ETL pipeline components
│   │   ├── 🔗 __init__.py
│   │   ├── data_ingestion.py             # MongoDB → CSV extraction & train/test split
│   │   ├── data_validation.py            # Schema validation & data drift detection
│   │   ├── data_transformation.py        # Feature engineering & KNN imputation
│   │   └── model_trainer.py              # Model training, hyperparameter tuning, serialization
│   │
│   ├── 🚀 pipeline/                      # High-level orchestration
│   │   ├── 🔗 __init__.py
│   │   ├── training_pipeline.py          # End-to-end training orchestration
│   │   └── batch_prediction.py           # Batch inference on new data
│   │
│   ├── 📡 cloud/                         # Cloud integration (AWS S3, etc.)
│   │   └── 🔗 __init__.py
│   │
│   ├── 🛠️ utils/                         # Utility functions
│   │   ├── 🔗 __init__.py
│   │   ├── main_utils/
│   │   │   ├── 🔗 __init__.py
│   │   │   └── utils.py                  # save_object, load_object, evaluate_models, etc.
│   │   └── ml_utils/
│   │       ├── 🔗 __init__.py
│   │       ├── metric/
│   │       │   ├── 🔗 __init__.py
│   │       │   └── classification_metric.py  # get_classification_score (F1, Precision, Recall)
│   │       └── model/
│   │           ├── 🔗 __init__.py
│   │           └── estimator.py          # NetworkModel (Preprocessor + Classifier wrapper)
│   │
│   ├── 📝 logging/                       # Logging configuration
│   │   ├── 🔗 __init__.py
│   │   └── logger.py                     # Structured logging setup
│   │
│   └── ⚠️ exception/                     # Custom exception handling
│       ├── 🔗 __init__.py
│       ├── exception.py                  # NetworkSecurityException class
│       └── logs/                         # Exception logs directory
│
├── 📂 Artifacts/                         # Generated pipeline artifacts
│   ├── data_ingestion/
│   │   ├── feature_store/
│   │   │   └── phisingData.csv
│   │   └── ingested/
│   │       ├── train.csv
│   │       └── test.csv
│   ├── data_validation/
│   │   ├── validated/
│   │   └── drift_report/
│   ├── data_transformation/
│   │   ├── transformed/
│   │   └── transformed_object/
│   └── model_trainer/
│       └── trained_model/
│
├── 📂 final_model/                       # Production model artifacts
│   ├── model.pkl                         # Trained classifier
│   └── preprocessor.pkl                  # KNN imputation pipeline
│
├── 📂 saved_models/                      # Alternative model storage
│   └── model.pkl
│
├── 📂 templates/                         # HTML templates for web UI
│   └── table.html                        # Prediction results display
│
├── 📂 prediction_output/                 # API prediction outputs
│   └── output.csv                        # Batch prediction results
│
├── 📂 valid_data/                        # Test data for validation
│   └── test.csv
│
├── 📂 logs/                              # Application logs
│
└── 📄 tree.txt                           # Project structure snapshot
```

**Key Directories Explained**:
- **networksecurity/**: Main Python package with modular components
- **Artifacts/**: Auto-generated pipeline outputs (data, models)
- **final_model/**: Production-ready serialized models
- **templates/**: Web UI templates for result visualization
- **data_schema/**: YAML-based data contracts for validation

---

## 🚀 Setup & Installation

### Prerequisites

- **Python Version**: 3.8 or higher
- **OS**: Linux, macOS, or Windows (all commands are cross-platform)
- **Memory**: Minimum 2GB RAM for model training
- **Internet**: Required for MongoDB and MLflow connectivity

### Environment Setup

#### 1. **Clone Repository**
```bash
git clone <repository-url>
cd networksecurity
```

#### 2. **Create Virtual Environment**

**Using `venv` (recommended for cross-platform)**:
```bash
# Create virtual environment
python -m venv venv

# Activate (Linux/macOS)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate
```

**Using `conda` (alternative)**:
```bash
# Create conda environment
conda create -n networksecurity python=3.10

# Activate
conda activate networksecurity
```

#### 3. **Install Dependencies**
```bash
# Upgrade pip
python -m pip install --upgrade pip setuptools wheel

# Install required packages
python -m pip install -r requirements.txt

# (Optional) Install the package in development mode
python -m pip install -e .
```

### Configuration

#### 1. **Environment Variables**

Create a `.env` file in the project root:
```bash
cat > .env << EOF
# MongoDB Connection
MONGODB_URL_KEY=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/?retryWrites=true&w=majority

# MLflow & DagShub (for experiment tracking)
MLFLOW_TRACKING_URI=https://dagshub.com/krishnanamdev558/networksecurity.mlflow
MLFLOW_TRACKING_USERNAME=<your-dagshub-username>
MLFLOW_TRACKING_PASSWORD=<your-dagshub-token>

# Optional: API Configuration
API_HOST=localhost
API_PORT=8000
EOF
```

**MongoDB Connection String**:
- Obtain from MongoDB Atlas: `https://cloud.mongodb.com`
- Format: `mongodb+srv://user:password@cluster.mongodb.net/?retryWrites=true&w=majority`
- Use Atlas certificate bundle (certifi handles automatically)

**DagShub Integration** (optional but recommended for MLOps):
- Create account: `https://dagshub.com`
- Generate personal token in Settings
- Update credentials in `.env`

#### 2. **Data Schema Configuration**

Edit [data_schema/schema.yaml](data_schema/schema.yaml) to match your data structure:
```yaml
columns:
  - having_IP_Address: int64
  - URL_Length: int64
  # ... (add all 31 features)
  - Result: int64  # Target variable

numerical_columns:
  - having_IP_Address
  - URL_Length
  # ... (all numerical features)
```

#### 3. **Load Initial Data to MongoDB**

Populate MongoDB with training data:
```bash
python -m networksecurity.components.data_ingestion push_data.py
# or use the provided utility
python push_data.py
```

This will:
1. Read [Network_Data/phisingData.csv](Network_Data/phisingData.csv)
2. Insert documents into MongoDB collection `NetworkData`
3. Database: `KRISHNA_AI`

---

## 📊 Execution Guide

### Option 1: Direct Python Script Execution

#### Train Full Pipeline

```bash
# Activate virtual environment
source venv/bin/activate  # Linux/macOS
# or
venv\Scripts\activate     # Windows

# Run end-to-end training
python -m main
```

**Output**:
- Trained model: `final_model/model.pkl`
- Preprocessor: `final_model/preprocessor.pkl`
- Artifacts directory: `Artifacts/`
- MLflow experiment logged (if configured)

#### Expected Output:
```
2025-01-15 10:23:45 - INFO - Initiate the data ingestion
2025-01-15 10:23:52 - INFO - Data Initiation Completed
2025-01-15 10:23:52 - INFO - Initiate the data Validation
2025-01-15 10:24:01 - INFO - data Validation Completed
2025-01-15 10:24:01 - INFO - Data Transformation started
2025-01-15 10:24:15 - INFO - Data Transformation Completed
2025-01-15 10:24:15 - INFO - Model Training started
2025-01-15 10:24:45 - INFO - Best model name: Random Forest, Best model score: 0.89
2025-01-15 10:24:47 - INFO - Model Training artifact created
```

---

### Option 2: FastAPI Web Service

#### Start the Inference Server

```bash
# Activate virtual environment
source venv/bin/activate  # Linux/macOS
# or
venv\Scripts\activate     # Windows

# Start FastAPI server
python -m app

# Server runs on: http://localhost:8000
```

**Console Output**:
```
INFO:     Uvicorn running on http://localhost:8000 (Press CTRL+C to quit)
INFO:     Application startup complete
```

#### Access API Documentation

- **Interactive Docs (Swagger UI)**: http://localhost:8000/docs
- **Alternative Docs (ReDoc)**: http://localhost:8000/redoc

---

### Option 3: Using Dockerfile (Production)

#### Build Container Image

```bash
# Build Docker image
docker build -t networksecurity:latest .

# Run container
docker run -p 8000:8000 \
  -e MONGODB_URL_KEY="<your-mongodb-url>" \
  -e MLFLOW_TRACKING_URI="<mlflow-uri>" \
  networksecurity:latest
```

---

## 📡 API Reference

### Endpoint 1: Train Model

**Request**:
```http
GET /train HTTP/1.1
Host: localhost:8000
```

**Response (Success)**:
```http
HTTP/1.1 200 OK
Content-Type: text/plain

Training is successful
```

**Response (Error)**:
```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "detail": "Error message details"
}
```

**Example Usage**:
```bash
# cURL
curl -X GET http://localhost:8000/train

# Python requests
import requests
response = requests.get('http://localhost:8000/train')
print(response.text)
```

---

### Endpoint 2: Make Predictions

**Request**:
```http
POST /predict HTTP/1.1
Host: localhost:8000
Content-Type: multipart/form-data

file=<CSV_FILE>
```

**Request Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | File (CSV) | Yes | CSV file with same schema as training data |

**CSV Format** (no target column needed):
```csv
having_IP_Address,URL_Length,Shortining_Service,...,Statistical_report
1,3,1,...,0
0,1,0,...,1
```

**Response (Success)**:
```html
<html>
  <table class="table table-striped">
    <thead>
      <tr>
        <th>having_IP_Address</th>
        <th>...</th>
        <th>Statistical_report</th>
        <th>predicted_column</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>1</td>
        <td>...</td>
        <td>0</td>
        <td>0</td>  <!-- Prediction -->
      </tr>
    </tbody>
  </table>
</html>
```

**Output File**:
- Predictions saved to: `prediction_output/output.csv`
- Contains original features + `predicted_column` (0=Legitimate, 1=Phishing)

**Example Usage**:
```bash
# cURL with file upload
curl -X POST http://localhost:8000/predict \
  -F "file=@test_data.csv"

# Python requests
import requests
with open('test_data.csv', 'rb') as f:
    files = {'file': f}
    response = requests.post('http://localhost:8000/predict', files=files)
print(response.text)
```

---

### Endpoint 3: Index / Documentation

**Request**:
```http
GET / HTTP/1.1
Host: localhost:8000
```

**Response**:
```http
HTTP/1.1 307 Temporary Redirect
Location: /docs
```

Redirects to interactive API documentation.

---

## ⚙️ Configuration

### Constants & Parameters

Located in [networksecurity/constant/training_pipeline/__init__.py](networksecurity/constant/training_pipeline/__init__.py):

```python
# Data Configuration
TARGET_COLUMN = "Result"                           # Binary classification target
FILE_NAME = "phisingData.csv"                      # Raw data file
PIPELINE_NAME = "NetworkSecurity"                  # Pipeline identifier
ARTIFACT_DIR = "Artifacts"                         # Output artifact directory

# MongoDB Configuration
DATA_INGESTION_COLLECTION_NAME = "NetworkData"     # Collection name
DATA_INGESTION_DATABASE_NAME = "KRISHNA_AI"        # Database name
DATA_INGESTION_TRAIN_TEST_SPLIT_RATIO = 0.2        # 80-20 split

# Data Transformation
DATA_TRANSFORMATION_IMPUTER_PARAMS = {
    "missing_values": np.nan,
    "n_neighbors": 3,                             # KNN neighbors
    "weights": "uniform",                         # Equal weight for all neighbors
}

# Model Training
MODEL_TRAINER_EXPECTED_SCORE = 0.6                # Minimum F1-score for deployment
MODEL_TRAINER_OVER_FITTING_UNDER_FITTING_THRESHOLD = 0.05
```

### Modifying Hyperparameters

Edit [networksecurity/components/model_trainer.py](networksecurity/components/model_trainer.py):

```python
def train_model(self, X_train, y_train, X_test, y_test):
    models = {...}
    
    params = {
        "Random Forest": {
            'n_estimators': [8, 16, 32, 128, 256],  # Adjust range
            'max_depth': [10, 20, None],            # Add new param
        },
        # ... modify other models similarly
    }
```

### Modifying Data Validation Thresholds

Edit [networksecurity/components/data_validation.py](networksecurity/components/data_validation.py):

```python
def detect_dataset_drift(self, base_df, current_df, threshold=0.05):  # Modify threshold
    # threshold: p-value significance level
    # Lower = stricter drift detection
```

---

## 📊 Monitoring & Logging

### Logging Configuration

Logs are generated in:
- **Console**: Real-time execution logs
- **File**: `networksecurity/exception/logs/` directory

**Log Levels**:
- `INFO`: Pipeline execution milestones
- `WARNING`: Non-critical issues
- `ERROR`: Failures and exceptions

**Accessing Logs**:
```bash
# View recent logs
tail -f networksecurity/exception/logs/error.log

# Search logs for specific errors
grep "ERROR" networksecurity/exception/logs/*.log
```

### MLflow Experiment Tracking

**View experiments locally** (if MLflow backend configured):
```bash
# Start MLflow UI
python -m mlflow ui --backend-store-uri ./mlruns

# Access UI: http://localhost:5000
```

**DagShub Remote Dashboard**:
- Navigate to: https://dagshub.com/krishnanamdev558/networksecurity
- View experiments, metrics, model history

### Model Performance Metrics

After training, review metrics in:
1. **Console Output**: Real-time F1, Precision, Recall scores
2. **MLflow**: Logged metrics per experiment run
3. **Artifacts**: Serialized `model.pkl` and `preprocessor.pkl`

---

## 🔍 Troubleshooting

### MongoDB Connection Issues

**Error**: `ServerSelectionTimeoutError`
```bash
# Solution 1: Verify connection string in .env
echo $MONGODB_URL_KEY

# Solution 2: Test connectivity
python -c "
import pymongo
client = pymongo.MongoClient('$MONGODB_URL_KEY')
print(client.server_info())
"
```

### Missing Data / NaN Values

**Error**: `ValueError: missing values encountered`
```bash
# Solution: Ensure DATA_TRANSFORMATION_IMPUTER_PARAMS are correctly set
# The KNN imputer should handle missing values automatically
```

### Model Performance Below Threshold

**Error**: `Best model score < 0.6`
```bash
# Solutions:
# 1. Check data quality in data_validation output
# 2. Adjust hyperparameter ranges in model_trainer.py
# 3. Verify target variable encoding (Result: 0 or 1)
# 4. Review data for class imbalance
```

### API Port Already in Use

**Error**: `Port 8000 is already in use`
```bash
# Solution: Use different port
python -m app --port 8001

# Or kill existing process
# Linux/macOS:
lsof -ti:8000 | xargs kill -9
# Windows:
netstat -ano | findstr :8000
taskkill /PID <PID> /F
```

---

## 📚 Additional Resources

- **Scikit-Learn Documentation**: https://scikit-learn.org
- **FastAPI Documentation**: https://fastapi.tiangolo.com
- **MLflow Documentation**: https://mlflow.org
- **MongoDB Atlas**: https://www.mongodb.com/cloud/atlas
- **DagShub**: https://dagshub.com

---

## 📝 License

[Add your license information here]

---

## ✨ Contributing

[Add contribution guidelines here]

---

**Project Maintainer**: Krishna Namdev

**Last Updated**: 2025-01-15

**Version**: 0.0.1
