# Weather Trend Forecasting

A data science assessment analyzing the Global Weather Repository dataset to explore weather trends and build a basic forecasting model.

## PM Accelerator Mission

By making industry-leading tools and education available to individuals from all backgrounds, PM Accelerator levels the playing field for future PM leaders. The organization grants aspiring and experienced Product Managers what they need most — access — by introducing them to industry leaders, surrounding them with the right PM ecosystem, and helping them discover the world of AI product management skills.



## Project Overview

This project analyzes daily weather readings for 248 cities worldwide, covering the period from May to September 2024. The goals were to:
1. Clean and prepare the raw dataset for analysis
2. Explore trends, correlations, and patterns through visualizations
3. Build and evaluate a basic forecasting model for temperature

## Dataset

- **Source:** [Global Weather Repository](https://www.kaggle.com/datasets/nelgiriyewithana/global-weather-repository/code) (Kaggle)
- **Size:** 21,322 rows × 41 columns (before cleaning)
- Not included in this repository — download it from the link above and place it in a `data/` folder before running the notebook.

## How to Run

1. Clone this repository
2. Install dependencies: `pip install -r requirements.txt`
3. Download the dataset from Kaggle (link above) and place `GlobalWeatherRepository.csv` inside a `data/` folder
4. Launch Jupyter from the project root: `jupyter notebook`
5. Open `notebooks/01_data_exploration.ipynb` and run all cells

## Data Cleaning

The dataset initially appeared complete — every column showed zero missing values. However, closer inspection revealed several issues that a simple missing-values check didn't catch:

| Issue | Finding | Action Taken |
|---|---|---|
| Disguised missing values | `air_quality_Carbon_Monoxide` and `air_quality_Sulphur_dioxide` each had 1 row with a `-9999` placeholder (physically impossible for a concentration reading) | Dropped the affected rows |
| Extreme outlier | One row recorded `wind_mph` of 1841.2 — faster than the speed of sound, and far beyond any real recorded wind speed | Dropped the row |
| Inconsistent categories | `condition_text` contained near-duplicate values differing only in capitalization (e.g., `'Partly Cloudy'` vs `'Partly cloudy'`) | Standardized to lowercase |
| Insufficient history per location | Readings per location ranged from 1 to 110 (median ~109). Locations with very few readings cannot support trend analysis | Filtered to locations with more than 20 readings (excluded 47 of 248 locations, retaining 96% of rows) |
| Duplicate rows | Checked directly | None found |
| Negative values elsewhere | Latitude/longitude negatives (southern/western hemispheres) and sub-zero temperatures | Confirmed valid, no action needed |

**Result:** 21,106 rows across 201 locations, each with a meaningful reading history.

## Exploratory Data Analysis

### Temperature trends
A single-location view (Kabul) showed a clear seasonal arc — rising from ~15-20°C in mid-May to a peak near 35-36°C in early August, then declining toward September.

Extending this to five climatically distinct locations (Kabul, Minsk, Beijing, Cairo, Helsinki) showed that:
- Cairo (desert climate) stayed consistently hot (30-42°C) with little seasonal variation in this window
- Kabul and Beijing showed a clear rise-then-fall seasonal pattern
- Minsk and Helsinki (continental/Nordic climates) stayed consistently cooler, with more day-to-day volatility

### Precipitation
Precipitation is fundamentally different from temperature — mostly zero, with occasional spikes — so a bar chart was used instead of a line chart, since connecting sparse rain events with a line would visually imply a false continuity between unrelated days.

Across the same five locations: Cairo showed negligible rainfall throughout (consistent with its desert climate), Kabul showed sparse, low-level events, while Minsk, Beijing, and Helsinki showed more frequent, larger rain events — up to ~5mm in Beijing's case.

### Correlations
A correlation heatmap across key weather variables (temperature, humidity, wind, pressure, UV index, cloud cover, visibility) revealed:
- Humidity and UV index: **-0.63** (the strongest relationship) — humid air tends to block UV radiation
- Humidity and cloud cover: **0.58** — more moisture generally accompanies more cloud
- Temperature and UV index: **0.54** — hotter days tend to be sunnier/clearer
- Wind speed and visibility showed little linear relationship with the other variables

Note: correlation does not imply causation — several of these relationships likely share an underlying cause (e.g., overall weather system type) rather than directly causing one another.

## Forecasting Model

**Approach:** Using Kabul as a case study, the data was split chronologically (88% train / 22% test) to simulate forecasting genuinely unseen future dates, rather than testing on data the model had already learned from.

**Model 1 — Linear Regression** (temperature as a function of days since the start of the dataset):

| Metric | Value |
|---|---|
| MAE | 7.79°C |
| RMSE | 8.63°C |
| R² | -7.90 |

The negative R² reveals a real limitation: linear regression can only fit a straight line, but the training period captured the *rising* half of a seasonal cycle. The model extrapolated that rise indefinitely, while the actual test period was past the seasonal peak and cooling — a pattern a straight line cannot represent.

**Model 2 — Polynomial Regression** (degree 2, allowing the model to fit a curve):

| Metric | Value |
|---|---|
| MAE | 4.64°C |
| RMSE | 5.48°C |
| R² | -2.58 |

Allowing the model to curve substantially improved accuracy (MAE improved by ~40%). However, R² remained negative, meaning both models still underperform a naive average-based prediction on this test window. This reflects a genuine, known challenge in time series forecasting: with less than one full seasonal cycle of training data, a model has limited ability to confidently extrapolate a turning point it has barely seen.

## Key Takeaways

- Data can appear "clean" by a basic missing-values check while still containing disguised errors (placeholder values, physically impossible outliers) — multiple types of checks are needed to catch different failure modes.
- Climate context (arid vs. continental vs. tropical) is visible directly in the data, both in temperature range and precipitation frequency.
- Model choice matters as much as model training: a technically well-trained linear model still fails structurally on curved, seasonal data.
- Honest evaluation which also includes negative results is part of a complete analysis.

## Project Structure
```
weather-trend-forecasting/
├── data/                          # raw CSV (not committed, see setup instructions)
├── notebooks/
│   └── 01_data_exploration.ipynb  # full analysis: cleaning, EDA, modeling
├── requirements.txt
├── .gitignore
└── README.md
```

## Author
[Raaga Priya Punaha] 