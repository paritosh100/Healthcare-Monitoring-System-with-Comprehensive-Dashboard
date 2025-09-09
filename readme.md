# Healthcare Monitoring System — Heart & Diabetes Risk Dashboard

A full-stack prototype that generates synthetic clinical data, trains ML models to predict **heart disease** and **diabetes** risk, and serves interactive dashboards for doctors, front‑desk staff, nurses, and patients. Built with **FastAPI**, **SQLite**, and **Plotly Dash**. fileciteturn0file0 fileciteturn0file1

---

## Why this exists
Heart disease and diabetes drive a huge share of preventable hospitalizations. Early detection matters, but real‑time context is rare. This system stitches together data, models, and role‑based dashboards so clinicians and patients can act before things escalate. fileciteturn0file0

---

## What’s inside
- **Relational data layer (SQLite)** with 5 normalized tables: `Patients`, `Appointments`, `LabReports`, `Vitals`, `RiskScores` fileciteturn0file1L356-L413  
- **FastAPI backend** exposing JSON endpoints for vitals, labs, appointments, demographics, and risk trends fileciteturn0file1L474-L593  
- **Dash frontends** (doctor, front desk, nurse, patient portal, reports, risk trend) with polling, charts, and exports fileciteturn0file1L622-L907  
- **Synthetic data generator** (Faker) with simple risk‑score formulas synced to visit timestamps fileciteturn0file1L415-L471  
- **Model training pipeline** (Linear Regression selected) + persisted `.pkl` artifacts for heart and diabetes risk fileciteturn0file0 fileciteturn0file1L684-L744

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
└─ requirements.txt
```
Names mirror the paper and slides. Adjust if your files differ. fileciteturn0file1L474-L907

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
(You can trim/expand as needed.)

### 2) Initialize the database
```bash
python backend/database_setup.py
```
Creates 5 normalized tables with indexes. fileciteturn0file1L318-L371

### 3) Generate synthetic data
```bash
python backend/data_generator.py --patients 10000 --visits 5
```
Seeds `Patients`, `Appointments`, `LabReports`, `Vitals`, and matching `RiskScores` per visit. fileciteturn0file1L415-L471

### 4) Train models
```bash
python backend/train_predictive_model.py
```
- Parses blood pressure into systolic/diastolic, builds feature matrix, trains 4 regressors, and **selects best by R²**  
- **Linear Regression** is the default winner:
  - Heart risk: **R² ≈ 0.4793**, **RMSE ≈ 0.0404**
  - Diabetes risk: **R² ≈ 0.1223**, **RMSE ≈ 0.1240**
- Saves: `models/heart_risk_model.pkl`, `models/diabetes_risk_model.pkl`  
fileciteturn0file0 fileciteturn0file1L744-L823

### 5) Run the API
```bash
uvicorn backend.backend:app --reload --port 8000
```
CORS is open in dev so Dash apps can hit the API. fileciteturn0file1L506-L573

### 6) Run dashboards (separate terminals)
```bash
python dashboards/dashboard_frontdesk.py
python dashboards/dashboard_doctor.py
python dashboards/patient_portal.py
python dashboards/risk_trend.py
python dashboards/reports.py
```
Each app polls the API on an interval for fresh data. fileciteturn0file1L622-L672

---

## API overview (FastAPI)

Base URL: `http://localhost:8000`

| Method | Route                               | What it returns |
|-------:|-------------------------------------|-----------------|
| GET    | `/active_patients`                  | Patients with `check_in_status='Checked-in'` fileciteturn0file1L551-L593 |
| GET    | `/patient_details/{patient_id}`     | Demographics + last visit date for one patient fileciteturn0file1L551-L593 |
| GET    | `/appointments_today`               | Today’s appointments joined with patient names fileciteturn0file1L593-L636 |
| GET    | `/recent_lab_reports`               | Recent 7‑day lab reports (limit 50) fileciteturn0file1L593-L636 |
| GET    | `/age_demographics`                 | Counts by age bucket (0–18, 19–35, 36–55, 55+) fileciteturn0file1L636-L680 |
| GET    | `/monthly_risk_trends`              | Monthly average heart/diabetes risk across cohort fileciteturn0file1L636-L680 |
| GET    | `/patient_risk_trend/{patient_id}`  | Monthly averages for a single patient fileciteturn0file1L636-L680 |
| POST   | `/save_risk`                        | Validates and inserts a new risk score row fileciteturn0file1L680-L720 |

---

## Dashboards

### Doctor
- Pick a patient, see current **heart/diabetes risk**, labeled low/moderate/high with colored progress bars
- Time‑series trend plot and quick context (age, gender, last visit)  
fileciteturn0file0 fileciteturn0file1L672-L745

### Front Desk
- Live KPIs: active patients, today’s appointments
- Age demographics, monthly risk trends, and recent lab activity  
fileciteturn0file0 fileciteturn0file1L745-L817

### Patient Portal
- Records, labs, meds, vaccines, messages, appointments, journal, goals (structure in place; wire as needed) fileciteturn0file1L817-L872

### Risk Trend
- Filter, drill into per‑patient risk trajectories; export history to CSV fileciteturn0file1L872-L907

### Reports
- Cohort overview: distributions, high‑risk table for fast triage fileciteturn0file1L907-L964

---

## Results at a glance
- **Models**: Linear Regression beat tree ensembles on this synthetic dataset
- **Performance**: Heart R² ≈ 0.48, Diabetes R² ≈ 0.12 (room to grow with richer features/tuning)  
- **Ops**: Dashboards surface high‑risk patients and rising trends clearly  
fileciteturn0file0 fileciteturn0file1L744-L823

---

## Limitations
Synthetic data and simplified risk formulas limit realism; dataset size is modest; polling can lag at scale. See the paper for concrete next steps (real EHRs, IoT streams, XAI, async/DB upgrades). fileciteturn0file0 fileciteturn0file1L964-L1182

---

## Roadmap
- Plug in de‑identified EHR datasets (e.g., MIMIC‑IV) and add more conditions
- Async routes + Postgres/MySQL; WebSockets for alerts
- Hyperparameter tuning (Optuna) and explainability (SHAP/LIME)
- Wearables integration via MQTT/WebSockets  
fileciteturn0file0 fileciteturn0file1L1002-L1120

---

## How to contribute
Small, focused PRs are best: tests for endpoints, pagination on read‑heavy views, and accessibility tweaks (contrast, keyboard navigation). fileciteturn0file1L1092-L1147

---

## Citation
If you reference this work in reports or coursework, please cite the **Final Research Report** and **Final Presentation** included in this repo. fileciteturn0file0 fileciteturn0file1
