# Delhi AQI Predictor — PM2.5 Forecasting for Okhla Phase-2

I built this project to predict Delhi's air quality using 6 years of real government sensor data from the Central Pollution Control Board (CPCB). This is not a tutorial project with toy data — it uses actual readings from a real monitoring station in Okhla, South Delhi.

---

## Why I Built This

Delhi's air quality is genuinely dangerous, especially between October and February. I wanted to build something that matters — a model that can predict tomorrow's PM2.5 levels so people can plan ahead, decide whether to wear a mask, or keep their kids indoors.

I started this project as a complete beginner. Everything here — the data pipeline, the cleaning, the models, the mistakes and fixes — I figured out step by step.

---

## What This Project Does

I built two machine learning models:

**Model 1** predicts the current day's PM2.5 concentration from other pollutant readings. It achieved R² = 0.9501 and MAE = 12.40 µg/m³, meaning it is off by only 12 µg/m³ on average across a range of 0–750.

**Model 2** predicts tomorrow's PM2.5 from today's readings using time series forecasting with lag features and rolling averages. It was trained on 2020–2024 data and tested on 2025–2026 data it had never seen.

---

## Data Source

I used the Okhla Phase-2 monitoring station in Delhi, operated by DPCC and reported by CPCB. The OpenAQ location ID for this station is 8239 — I found this directly from the URL when visiting the station page on explore.openaq.org.

The data covers January 2020 to February 2026, giving me around 1,008 usable days after cleaning. The parameters available include PM2.5, PM10, NO2, SO2, CO, O3, NOx, temperature, humidity, and wind data.

I did not use the OpenAQ API to download this data. The API has a rate limit of 60 requests per minute and 2,000 per hour. Downloading 6 years of data would have required ~17,500 API calls and around 9 hours of continuous downloading — with a risk of getting permanently banned. Instead, I used OpenAQ's public AWS S3 bucket which has no rate limits, no API key required, and downloads the same data in minutes.

---

## Project Structure

```
delhi_okhla_project/
│
├── raw_data/                            # Downloaded .csv.gz files from OpenAQ S3
│   ├── 2020/
│   ├── 2021/
│   ├── 2022/
│   ├── 2023/
│   ├── 2024/
│   ├── 2025/
│   └── 2026/
│
├── delhi_newdelhi_2020_2026_clean.csv   # Final cleaned dataset (auto-generated)
├── okhla_pm25_FINAL_best_model.pkl      # Saved trained model (auto-generated)
├── model_features.json                  # Feature list for the saved model
│
├── okhla_aqi_model.ipynb                # Main Jupyter notebook — full pipeline
├── requirements.txt                     # All Python dependencies
└── README.md                            # This file
```

---

## Setup Instructions

### Step 1 — Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/delhi-aqi-predictor.git
cd delhi-aqi-predictor
```

### Step 2 — Create a Virtual Environment

```bash
python -m venv venv
source venv/bin/activate
```

### Step 3 — Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 4 — Install and Configure AWS CLI

I used AWS CLI to download data from OpenAQ's public S3 bucket. No AWS account is needed — the bucket is completely public.

```bash
# Install on Mac
brew install awscli

# Must be set to us-east-1 — this is where OpenAQ's bucket lives
aws configure set region us-east-1
```

### Step 5 — Download the Data

I downloaded data year by year. Each year takes 1–3 minutes depending on internet speed.

```bash
mkdir -p raw_data/{2020,2021,2022,2023,2024,2025,2026}

aws s3 cp s3://openaq-data-archive/records/csv.gz/locationid=8239/year=2020/ ./raw_data/2020/ --recursive --no-sign-request
aws s3 cp s3://openaq-data-archive/records/csv.gz/locationid=8239/year=2021/ ./raw_data/2021/ --recursive --no-sign-request
aws s3 cp s3://openaq-data-archive/records/csv.gz/locationid=8239/year=2022/ ./raw_data/2022/ --recursive --no-sign-request
aws s3 cp s3://openaq-data-archive/records/csv.gz/locationid=8239/year=2023/ ./raw_data/2023/ --recursive --no-sign-request
aws s3 cp s3://openaq-data-archive/records/csv.gz/locationid=8239/year=2024/ ./raw_data/2024/ --recursive --no-sign-request
aws s3 cp s3://openaq-data-archive/records/csv.gz/locationid=8239/year=2025/ ./raw_data/2025/ --recursive --no-sign-request
aws s3 cp s3://openaq-data-archive/records/csv.gz/locationid=8239/year=2026/ ./raw_data/2026/ --recursive --no-sign-request
```

Verify the download:

```bash
find ./raw_data -name "*.csv.gz" | wc -l
# I got 1007 files — missing days are normal due to sensor downtime
```

### Step 6 — Run the Notebook

```bash
jupyter notebook
```

Open `okhla_aqi_model.ipynb` and run all cells in order from top to bottom.

---

## My ML Pipeline

```
1,007 daily .csv.gz files from OpenAQ S3
            ↓
Read and combine all files using glob + pandas
            ↓
Convert timestamps from UTC to IST (UTC +5:30)
            ↓
Calculate daily average for each pollutant
            ↓
Pivot to wide format — one column per pollutant, one row per day
            ↓
Feature engineering
    → month, year, day_of_year
    → is_winter flag (October–February = 1, else = 0)
    → Lag features: pm25_lag1, lag2, lag3, lag7
    → Rolling averages: 7-day, 14-day
            ↓
Handle missing values
    → Columns over 60% missing → dropped entirely
    → Columns under 5% missing → filled with median
            ↓
Train / Test Split
    → Model 1: random 80/20 split
    → Model 2: chronological — 2020–2024 train, 2025–2026 test
            ↓
Train Random Forest Regressor
    → n_estimators=200, max_depth=15, min_samples_leaf=3
            ↓
Results: R² = 0.9501, MAE = 12.40 µg/m³
```

---

## Results

### Model 1 — Same Day PM2.5 Prediction

| Metric | Value |
|---|---|
| R² Score | 0.9501 |
| Mean Absolute Error | 12.40 µg/m³ |
| Algorithm | Random Forest Regressor |

### Model 2 — Tomorrow's PM2.5 (Time Series)

I trained on 2020–2024 and tested on 2025–2026 data the model had never seen. The model correctly captured winter spikes every October–January, clean monsoon air every July–September, and day-to-day momentum in pollution levels.

---

## Algorithms I Tried

I did not just go with the first algorithm that worked. I tested three approaches and compared them honestly.

| Algorithm | MAE | R² | Notes |
|---|---|---|---|
| Random Forest | 12.40 | 0.9501 | Winner — stable on small datasets |
| XGBoost default | 13.18 | 0.9379 | Worse — needs more data to shine |
| XGBoost tuned via GridSearchCV | 12.87 | 0.9472 | Improved after tuning, still lost |

Random Forest won. With around 1,000 rows, its approach of building trees independently generalised better than XGBoost's sequential correction, which needs more data to outperform simpler methods.

---

## Mistakes I Made and What I Learned

### Data Leakage

During feature engineering I created a feature called `pm25_pm10_ratio = pm25 / pm10`. This accidentally included the target variable (PM2.5) inside the input, so the model was essentially reading the answer from the question paper. It showed a fake R² of 0.9913. After removing it, the honest score dropped to 0.9425 — worse than the original baseline.

The rule I follow now: before adding any feature, I ask "would I have this value at prediction time without already knowing the answer?" If no, it is leakage and it gets removed.

### Chronological Splitting for Time Series

For the tomorrow's prediction model, a random split cannot be used. If 2026 data ends up in the training set and 2022 data ends up in the test set, the model is learning from the future. I used a hard date cutoff — everything before 2025 trains the model, everything from 2025 onwards tests it.

### Missing Data from Sensor Downtime

Several columns (NOx, wind speed, wind direction, humidity, temperature) were 60–87% empty. This is normal with real CPCB data — sensors go offline for maintenance, power cuts, and calibration. I dropped these columns rather than filling them, because imputing 87% of a column introduces more noise than signal.

---

## AQI Category Reference (India — CPCB Scale)

| PM2.5 (µg/m³) | Category | Advisory |
|---|---|---|
| 0 – 30 | Good | Safe for everyone |
| 31 – 60 | Satisfactory | Sensitive groups be cautious |
| 61 – 90 | Moderate | Limit prolonged outdoor activity |
| 91 – 120 | Poor | Wear N95 mask outside |
| 121 – 250 | Very Poor | Avoid going outside |
| 250+ | Severe | Stay indoors, seal windows |

---

## Using the Saved Model

Once the notebook has been run, the trained model is saved as a `.pkl` file and can be loaded and used directly:

```python
import joblib, pandas as pd

model = joblib.load('okhla_pm25_FINAL_best_model.pkl')

# Predict PM2.5 for a Delhi winter day
winter_day = pd.DataFrame([{
    'co': 900, 'no2': 95, 'o3': 18,
    'pm10': 350, 'so2': 20,
    'month': 11, 'year': 2025,
    'day_of_year': 315, 'is_winter': 1
}])

prediction = model.predict(winter_day)[0]
print(f"Predicted PM2.5: {prediction:.1f} µg/m³")
# Output: 201.7 µg/m³ — Very Poor
```

---

## Troubleshooting

**AWS download returns access denied or no such bucket:**
Make sure the region is set to us-east-1 before running the download commands.

```bash
aws configure set region us-east-1
```

**GradientBoostingRegressor throws a NaN error:**
Switch to `HistGradientBoostingRegressor` — it handles missing values natively and is the recommended modern version.

```python
from sklearn.ensemble import HistGradientBoostingRegressor
```

**File count is lower than expected after download:**
I got 1,007 files instead of ~2,000. This is expected. CPCB sensors go offline frequently due to maintenance and power cuts. Missing days are a feature of real government data, not a download error.

**"Feb 24 not found" during the time series prediction cell:**
This is expected. When I created the tomorrow target using `shift(-1)`, the last row loses its target value since there is no day after the final date. The last usable row in `ts_df` is always one day before the last date in `final_df`. Use `final_df` to look up the most recent actual reading.

---

## Data Attribution

Data sourced from OpenAQ (openaq.org), which aggregates measurements from CPCB (Central Pollution Control Board), Government of India. The Okhla Phase-2 station is operated by DPCC (Delhi Pollution Control Committee).

OpenAQ Terms of Use: https://openaq.org/terms
