# Week 2 — Data Wrangling & Insight Reporting

**Author:** Alex Kariuki
**Course:** Data Science Bootcamp — Week 2 Assignment
**Dataset:** `ops_sensor_log_dirty.csv` — one week of sensor logs from a fictional processing plant

---

## What this project does

This assignment takes a deliberately messy sensor dataset and walks through the full data
wrangling lifecycle: profiling the quality issues, building a reusable cleaning pipeline,
running time-series analysis, and communicating the findings to a non-technical audience.

---

## Repository contents

| File | Description |
|------|-------------|
| `ops_sensor_log_dirty.csv` | Raw dataset — 5,015 rows of pressure, temperature, and flow rate readings with intentional quality issues |
| `week2_data_wrangler.ipynb` | Main Jupyter notebook covering all 5 sections of Part A |
| `Week2_Insight_Report_AlexKariuki.pdf` | 1-page annotated chart report for Part B (written for a Plant Manager audience) |
| `raw_vs_cleaned_pressure.png` | Key visualisation: raw vs cleaned Pressure PSI over time |
| `timeseries_rolling.png` | Hourly pressure and temperature with 24-hour rolling average |
| `heatmap_zone_shift.png` | Mean pressure heatmap broken down by Zone × Shift |
| `missing_values_bar.png` | Missing value distribution across columns |

---

## Notebook structure (Part A)

### 1. Ingestion & Profiling
Loads the raw CSV, runs `.info()`, `.describe()`, and a `missingno` bar chart to get a
feel for the data. Wraps up with a **Data Health Report** in Markdown identifying 5 quality issues:

- Missing values across Zone, Shift, Pressure, Temperature, and Flow Rate (~1% each)
- 15 exact duplicate rows
- 15 inconsistent Zone name variants (`Zone_North`, `ZONE-NORTH`, `z_north`, etc.)
- Physically impossible sensor readings (Pressure: −50 to 15,000 PSI; Temperature: −273 to 1,500 °C)
- Timestamp stored as a plain string rather than `datetime64`

### 2. Cleaning Pipeline
A single reusable function `clean_ops_data(df)` that:
- Parses timestamps with `pd.to_datetime()` (handles mixed formats gracefully)
- Drops exact duplicates
- Standardises all Zone name variants to 5 canonical names via keyword matching
- Normalises Shift to title-case
- **Interpolates** missing numeric sensor values using `method='time'` — chosen over mean/median fill because sensors emit continuous physical signals, and interpolating between neighbours preserves the real signal shape
- **Forward-fills** missing Zone and Shift values — these don't change randomly between readings
- Filters out readings outside physically defensible ranges (Pressure 0–500 PSI, Temperature −10–200 °C, Flow 0–2000 LPM)

### 3. Time-Series Analysis
- Resamples the cleaned data to **hourly frequency** (mean aggregation)
- Calculates a **24-hour rolling average** for Pressure and Temperature
- Plots both metrics: hourly readings + rolling trend line

### 4. Aggregation
- Mean, Max, Min for every sensor metric grouped by **Shift** (Morning / Afternoon / Night)
- Same summary grouped by **Zone** (North / South / East / West / Central)
- Combined **Zone × Shift heatmap** for mean Pressure

### 5. Visualisation: Raw vs Cleaned
Side-by-side comparison of raw and cleaned Pressure PSI. The raw panel shows the distorted
Y-axis caused by 15,000 PSI outliers. The clean panel reveals a **consistent nightly pressure
rise (00:00–02:00)** that was completely buried in the noise before cleaning.

---

## PDF Report (Part B)

`Week2_Insight_Report_AlexKariuki.pdf` is a 1-page report written for a Plant Manager —
no code, clear language, three paragraphs:

- **Context:** What was wrong with the raw data and why it couldn't be trusted
- **Insight:** The nightly pressure spike in North and West Zones that only became visible after cleaning
- **Action:** Three specific operational recommendations — inspect the overnight window, set automated alerts, repair faulty sensors

---

## How to run

```bash
# Clone the repo
git clone https://github.com/kariukialex999/week2-alex-kariuki.git
cd week2-alex-kariuki

# Install dependencies (assumes conda)
conda activate data-science
# or: pip install pandas numpy matplotlib missingno reportlab nbformat

# Run the notebook
jupyter notebook week2_data_wrangler.ipynb
```

All cells run top-to-bottom with no manual steps. The notebook will regenerate all chart
images and the cleaned dataset in memory.

---

## Key finding

The raw data made the plant look noisy but stable — the inflated mean pressure (255 PSI)
masked the real operating level. After cleaning, the true mean drops to ~200 PSI and a
**repeating nightly pressure elevation of 20–30 PSI between midnight and 02:00** becomes
clearly visible. North Zone and West Zone show the sharpest rise. This pattern repeats
every day of the week studied and warrants immediate investigation.

---

## Part C — Peer Review

To review a peer's submission:
1. Clone their repository
2. Replace `ops_sensor_log_dirty.csv` with the shared dataset (or use theirs directly)
3. Run `clean_ops_data(df)` from their notebook on the dataset
4. Open a GitHub Issue on their repo with feedback on:
   - One thing their cleaning logic handled well
   - One potential edge case their code might miss
   - Clarity of their variable names
