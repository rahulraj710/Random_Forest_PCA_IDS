# Network Intrusion Detection System
 
A machine learning-based network intrusion detection system (IDS) trained on the NSL-KDD dataset. The project compares multiple Random Forest configurations and dimensionality reduction strategies, then deploys the best-performing model as a REST API using FastAPI.
 
## Overview
 
Network intrusion detection is the problem of classifying network traffic as either normal or malicious. This project explores how dimensionality reduction techniques affect model performance and deploys the results as a production-ready API with single-record and bulk prediction endpoints.
 
## Project Structure
 
```
network-intrusion-detection/
├── notebooks/
│   ├── baseline_rf_model.ipynb    # Random Forest with full features and top-N feature selection
│   └── PCA.ipynb                  # PCA-based dimensionality reduction + Random Forest
├── data/
│   └── KDDTest+.txt               # NSL-KDD dataset (txt files)
│   └── KDDTrain+.txt 
├── api/
│   ├── main.py                    # FastAPI app and endpoint definitions
│   ├── model.py                   # Model loading logic
│   ├── schemas.py                 # Pydantic request/response models
│   ├── preprocessing.py           # Preprocessing pipeline (scaling, encoding)
│   └── models/                    # Serialized model and preprocessor (generated locally)
│       ├── rf_model.joblib
│       └── preprocessor.joblib
├── requirements.txt
├── .gitignore
└── README.md
```
 
## Models Compared
 
| Model | Description | Features |
|---|---|---|
| A | Random Forest — full features | 122 (after one-hot encoding) |
| B | Random Forest — top-N feature selection | 10 / 15 / 20 / 30 |
| C | PCA + Random Forest | 88 components (95% variance) |
 
**Key finding:** Model A (full features) achieved the best performance. PCA increased training time and slightly reduced accuracy, while top-N feature selection performed better than PCA but worse than the full feature baseline. This result reflects a fundamental mismatch — PCA optimizes for variance, not class separation, and Random Forest handles correlated features natively without needing dimensionality reduction.
 
## Dataset
 
This project uses the [NSL-KDD dataset](https://www.unb.ca/cic/datasets/nsl.html), an improved version of the KDD Cup 1999 dataset for network intrusion detection research.
 
- 41 raw features → 122 features after one-hot encoding (`protocol_type`, `service`, `flag`)
- Binary classification: `normal` vs. `attack`
- 80/20 stratified train/test split
> Note: NSL-KDD is a well-studied benchmark dataset. Near-perfect accuracy (~99.9%) is expected and consistent with published literature. The dataset is best used for methodology comparison rather than as a reflection of real-world detection difficulty.
 
## API
 
The REST API exposes two prediction endpoints, documented automatically via Swagger UI.
 
### Endpoints
 
**`POST /predict`** — Single record prediction
 
```json
Request:
{
  "duration": 0,
  "protocol_type": "tcp",
  "service": "http",
  "flag": "SF",
  "src_bytes": 181,
  ...
}
 
Response:
{
  "prediction": "normal",
  "confidence": 0.97
}
```
 
**`POST /predict/bulk`** — Bulk prediction
 
```json
Request:
{
  "records": [ { ... }, { ... } ]
}
 
Response:
{
  "predictions": ["normal", "attack", "normal"]
}
```
 
### Swagger UI
 
Once running, visit `http://localhost:8000/docs` for the interactive API documentation.
 
## Getting Started
 
### Prerequisites
 
- Python 3.9+
- pip
### Installation
 
```bash
git clone https://github.com/yourusername/network-intrusion-detection.git
cd network-intrusion-detection
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```
 
### Generate the Model
 
Run the training notebook first to serialize the model and preprocessing pipeline:
 
```bash
jupyter notebook notebooks/baseline_rf_model.ipynb
```
 
This will generate `rf_model.joblib` and `preprocessor.joblib` inside `api/models/`. These files are excluded from version control and must be generated locally.
 
### Run the API
 
```bash
cd api
uvicorn main:app --reload
```
 
The API will be available at `http://localhost:8000`.
 
## Technologies
 
- Python 3.9+
- scikit-learn — Random Forest, PCA, preprocessing pipeline
- FastAPI — REST API framework
- Pydantic — request/response validation
- joblib — model serialization
- uvicorn — ASGI server
- pandas, numpy — data processing
- Jupyter — experimentation and analysis
 
