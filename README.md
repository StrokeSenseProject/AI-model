# StrokeSense AI API Package

## Overview

StrokeSense is an early screening support API for stroke risk estimation using tabular health data.
This package is intended for backend/fullstack integration.

Important: StrokeSense is not a medical diagnosis and must not replace consultation with qualified healthcare professionals.

## Main Files for Fullstack

- app.py: FastAPI inference service.
- requirements.txt: pinned Python dependencies.
- strokesense_model.keras: Keras model artifact.
- strokesense_saved_model/: TensorFlow SavedModel fallback artifact.
- threshold.json: decision threshold for Low Risk / High Risk.
- feature_schema.json: model input schema.
- feature_mapping.json: source of truth for processed feature order.
- model_metadata.json: model and input metadata.
- preprocessor.pkl: transforms raw user input into 18 processed model features.

## API Endpoints

- GET /health
- POST /predict

## How to Run

Install dependencies:

```bash
pip install -r requirements.txt
```

Run API:

```bash
uvicorn app:app --host 0.0.0.0 --port 8000
```

Check health:

```bash
curl http://localhost:8000/health
```

Expected healthy response:

```json
{
  "status": "ok",
  "model_loaded": true,
  "processed_mode_enabled": true,
  "raw_mode_enabled": true
}
```

## Input Modes

### 1. Raw Mode

Recommended for backend/frontend integration.
The backend sends 8 user-facing health fields. The API uses preprocessor.pkl to transform them into 18 processed model features.

Example request:

```json
{
  "mode": "raw",
  "raw_features": {
    "age": 55,
    "hypertension": 1,
    "heart_disease": 0,
    "ever_married": "Yes",
    "work_type": "Private",
    "avg_glucose_level": 180,
    "bmi": 29.5,
    "smoking_status": "formerly smoked"
  }
}
```

### 2. Processed Mode

Use this only if backend already has 18 processed numeric features in the exact order from feature_mapping.json.

Example request:

```json
{
  "mode": "processed",
  "features": [0.1, -0.2, 0.3, 0, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0, 1, 0, 1, 0]
}
```

## Example Response

```json
{
  "stroke_probability": 0.42,
  "threshold": 0.37,
  "prediction": 1,
  "risk_label": "High Risk",
  "disclaimer": "This prediction is for early screening support only and is not a medical diagnosis. Please consult a qualified healthcare professional for medical advice.",
  "input_mode": "raw",
  "feature_order_source": "feature_mapping.json"
}
```

## Output Meaning

- stroke_probability: model score for stroke risk screening.
- threshold: cutoff used to convert probability into Low Risk or High Risk.
- prediction: 0 means Low Risk, 1 means High Risk.
- risk_label: user-facing risk category.
- disclaimer: mandatory medical disclaimer.

## Deployment Notes

- app.py first tries to load strokesense_model.keras.
- If .keras loading fails, app.py falls back to strokesense_saved_model/.
- Keep strokesense_saved_model/ as a complete folder. Do not move only saved_model.pb without variables/.
- preprocessor.pkl is required for raw mode.
- Do not send target or stroke to /predict.
- Use a fresh virtual environment when installing requirements.txt.

## Model Metrics

- Final threshold: 0.37
- Accuracy: 0.7348336594911937
- ROC-AUC: 0.830843621399177
- PR-AUC: 0.27160124203746244
- Precision: 0.132890365448505
- Recall: 0.8
- F1-score: 0.22792022792022792
- Specificity: 0.7314814814814815
- Balanced accuracy: 0.7657407407407408
- Confusion matrix: TN=711, FP=261, FN=10, TP=40

Accuracy alone is not sufficient because the dataset is highly imbalanced.
For this screening use case, recall, PR-AUC, precision, F1-score, specificity, balanced accuracy, and confusion matrix are more informative.

## Medical Disclaimer

StrokeSense is for early screening support only and is not a medical diagnosis.
It is not a substitute for consultation with qualified healthcare professionals.
Users with high-risk results or stroke symptoms should seek medical care.