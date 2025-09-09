# Healthcare Monitoring System — Heart & Diabetes Risk Dashboard

A full-stack prototype that generates synthetic clinical data, trains ML models to predict **heart disease** and **diabetes** risk, and serves interactive dashboards for doctors, front‑desk staff, nurses, and patients. Built with **FastAPI**, **SQLite**, and **Plotly Dash**. 

---

## Why this exists
Heart disease and diabetes drive a huge share of preventable hospitalizations. Early detection matters, but real‑time context is rare. This system stitches together data, models, and role‑based dashboards so clinicians and patients can act before things escalate. 

---

## What’s inside
- **Relational data layer (SQLite)** with 5 normalized tables: `Patients`, `Appointments`, `LabReports`, `Vitals`, `RiskScores` 
- **FastAPI backend** exposing JSON endpoints for vitals, labs, appointments, demographics, and risk trends 
- **Dash frontends** (doctor, front desk, nurse, patient portal, reports, risk trend) with polling, charts, and exports 
- **Synthetic data generator** (Faker) with simple risk‑score formulas synced to visit timestamps 
- **Model training pipeline** (Linear Regression selected) + persisted `.pkl` artifacts for heart and diabetes risk 

---

## Repo layout (suggested)
```
.
├─ backend/
│  ├─ backend.py                 # FastAPI app & routes
│  ├─ database_setup.py          # creates tables
│  ├─ data_generator.py          # Faker-based seeding
│  ├─ train_predictive_model.py  # ML pipeline, saves .pkl
│  ├─ login_utils.py             # simple auth utilities
│  └─ healthcare.db              # SQLite (generated)
├─ dashboards/
│  ├─ dashboard_doctor.py
│  ├─ dashboard_frontdesk.py
│  ├─ nurse_dashboard.py         # if split out; optional
│  ├─ patient_portal.py
│  ├─ risk_trend.py
│  └─ reports.py
├─ models/
│  ├─ heart_risk_model.pkl
│  └─ diabetes_risk_model.pkl
├─ README.md
```
Names mirror the paper and slides. Adjust if your files differ. 

---

## Quickstart

### 1) Environment
```bash
python -m venv .venv
source .venv/bin/activate   # on Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

`requirements.txt` (baseline):
```
fastapi
uvicorn
pydantic
requests
python-multipart
plotly
dash
dash-bootstrap-components
pandas
numpy
scikit-learn
xgboost
joblib
faker
```

### 2) Initialize the database
```bash
python backend/database_setup.py
```
Creates 5 normalized tables with indexes. 

### 3) Generate synthetic data
```bash
python backend/data_generator.py --patients 10000 --visits 5
```
Seeds `Patients`, `Appointments`, `LabReports`, `Vitals`, and matching `RiskScores` per visit. 

### 4) Train models
```bash
python backend/train_predictive_model.py
```
- Parses blood pressure into systolic/diastolic, builds feature matrix, trains 4 regressors, and **selects best by R²**  
- **Linear Regression** is the default winner:
  - Heart risk: **R² ≈ 0.4793**, **RMSE ≈ 0.0404**
  - Diabetes risk: **R² ≈ 0.1223**, **RMSE ≈ 0.1240**
- Saves: `models/heart_risk_model.pkl`, `models/diabetes_risk_model.pkl`  


### 5) Run the API
```bash
uvicorn backend.backend:app --reload --port 8000
```
CORS is open in dev so Dash apps can hit the API. 

### 6) Run dashboards (separate terminals)
```bash
python dashboards/dashboard_frontdesk.py
python dashboards/dashboard_doctor.py
python dashboards/patient_portal.py
python dashboards/risk_trend.py
python dashboards/reports.py
```
Each app polls the API on an interval for fresh data. 

---

## API overview (FastAPI)

Base URL: `http://localhost:8000`

| Method | Route                               | What it returns |
|-------:|-------------------------------------|-----------------|
| GET    | `/active_patients`                  | Patients with `check_in_status='Checked-in'` |
| GET    | `/patient_details/{patient_id}`     | Demographics + last visit date for one patient |
| GET    | `/appointments_today`               | Today’s appointments joined with patient names |
| GET    | `/recent_lab_reports`               | Recent 7‑day lab reports (limit 50) |
| GET    | `/age_demographics`                 | Counts by age bucket (0–18, 19–35, 36–55, 55+) |
| GET    | `/monthly_risk_trends`              | Monthly average heart/diabetes risk across cohort |
| GET    | `/patient_risk_trend/{patient_id}`  | Monthly averages for a single patient |
| POST   | `/save_risk`                        | Validates and inserts a new risk score row |

---

## Dashboards

### Doctor
- Pick a patient, see current **heart/diabetes risk**, labeled low/moderate/high with colored progress bars
- Time‑series trend plot and quick context (age, gender, last visit)  

### Front Desk
- Live KPIs: active patients, today’s appointments
- Age demographics, monthly risk trends, and recent lab activity  

### Patient Portal
- Records, labs, meds, vaccines, messages, appointments, journal, goals (structure in place; wire as needed) 

### Risk Trend
- Filter, drill into per‑patient risk trajectories; export history to CSV 

### Reports
- Cohort overview: distributions, high‑risk table for fast triage

---

## Results at a glance
- **Models**: Linear Regression beat tree ensembles on this synthetic dataset
- **Performance**: Heart R² ≈ 0.48, Diabetes R² ≈ 0.12 (room to grow with richer features/tuning)  
- **Ops**: Dashboards surface high‑risk patients and rising trends clearly  

---

## Limitations
Synthetic data and simplified risk formulas limit realism; dataset size is modest; polling can lag at scale. See the paper for concrete next steps (real EHRs, IoT streams, XAI, async/DB upgrades). 

---

## Roadmap
- Plug in de‑identified EHR datasets (e.g., MIMIC‑IV) and add more conditions
- Async routes + Postgres/MySQL; WebSockets for alerts
- Hyperparameter tuning (Optuna) and explainability (SHAP/LIME)
- Wearables integration via MQTT/WebSockets  

---

## How to contribute
Small, focused PRs are best: tests for endpoints, pagination on read‑heavy views, and accessibility tweaks (contrast, keyboard navigation). 

---

## Citation
If you reference this work in reports or coursework, please cite the **Final Research Report** and **Final Presentation** included in this repo. 
