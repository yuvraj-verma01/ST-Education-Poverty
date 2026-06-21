# DSM Final Project: ST Education and Livelihood Analysis

A data-driven policy analysis platform for understanding the overlapping education disadvantage and economic vulnerability of Scheduled Tribes (ST) across Indian states. The project ingests 17 government datasets, builds a unified state-level analytical database, and exposes findings through an interactive Streamlit dashboard with an LLM-powered SQL analyst.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Repository Structure](#2-repository-structure)
3. [Raw Data Sources](#3-raw-data-sources)
4. [Data Build Pipeline](#4-data-build-pipeline)
5. [Analysis Outputs](#5-analysis-outputs)
6. [Dashboard](#6-dashboard)
7. [LLM Engine](#7-llm-engine)
8. [Database Analyzer App](#8-database-analyzer-app)
9. [EDA and Question Scripts](#9-eda-and-question-scripts)
10. [Setup and Installation](#10-setup-and-installation)
11. [Running the Project](#11-running-the-project)
12. [Research Questions](#12-research-questions)
13. [Priority State Rankings](#13-priority-state-rankings)

---

## 1. Project Overview

India's Scheduled Tribe population faces compounding disadvantages in both education and livelihood. This project systematically combines 17 disaggregated government datasets — covering literacy, enrolment, dropout, gender parity, poverty, employment, MGNREG, scholarships, and tribal village geography — into a single reproducible state-level analysis framework.

The primary analytical lens is identifying the 19 high-ST-share states where educational exclusion and economic vulnerability co-occur, and ranking them by an evidence-based priority score. All findings are interactively explorable through a Streamlit dashboard that lets users drill into individual research questions, compare states on any pair of variables, view choropleth maps, and run natural-language queries against the underlying database using an LLM-powered SQL analyst.

**Key numbers from the build pipeline:**
- 17 cleaned datasets merged
- 34 states and union territories in the combined table
- 19 high-ST-share states in the priority analysis
- ~200 columns in the master state-level dataset
- 13 structured research questions with dedicated visualizations

---

## 2. Repository Structure

```text
data/
  raw/                          17 source CSV files (tracked, short filenames)
scripts/
  build_project_data.py         Main data pipeline — cleans, merges, scores, exports
  run_policy_eda.py             EDA, clustering, and correlation analysis
  create_eda_notebook.py        Auto-generates the EDA Jupyter notebook
  generate_q2_graphs.py         Question-specific chart generators (q2, q4, q6, q8, q10, q12)
  generate_q4_graphs.py
  generate_q6_graphs.py
  generate_q8_graphs.py
  generate_q10_graphs.py
  generate_q12_graphs.py
dashboard_app/
  streamlit_app.py              Main Streamlit dashboard application
  requirements.txt              Dashboard-specific Python dependencies
  india_state.geojson           India state boundary geometry for choropleth maps
database/
  app.py                        Standalone Streamlit SQL analyzer (MySQL)
  dsm_project.ipynb             Database schema notebook
outputs/
  cleaned/                      17 normalized CSV datasets
  analysis/                     State-level master tables, scores, correlations
  eda/                          EDA report tables and figures
  figures/                      Core summary figures from the build pipeline
  st_education_project.sqlite   SQLite database (all 17 tables + state analysis)
graphs/
  q2/ q4/ q6/ q8/ q10/ q12/    Question-specific chart outputs (PNG)
notebooks/
  eda_policy_analysis.ipynb     Comprehensive EDA notebook
  interactive_India_dash.ipynb  Interactive analysis notebook
Additional(Potential)Data/      Extra datasets not in main pipeline
```

---

## 3. Raw Data Sources

All source data is stored under `data/raw/` as short-named CSV copies. The 17 datasets and what they contain:

| File | Contents |
|---|---|
| `demographics_st.csv` | ST population, total population, ST share %, decadal growth by state |
| `literacy.csv` | ST literacy rate, total literacy rate, literacy gap, female breakdown |
| `enrolment_st.csv` | ST enrolment by level (pre-primary → secondary), female share % |
| `ger_st.csv` | Gross Enrolment Ratio by class group and gender (2020-22) |
| `ger_st_latest.csv` | Latest GER data (2023-24), Gender Parity Index variants |
| `gpi_st.csv` | Gender Parity Index at secondary and higher-secondary levels (2020-22) |
| `dropout_st.csv` | Dropout rates at primary, upper-primary, and secondary levels (2021-22) |
| `poverty_st.csv` | ST Below Poverty Line % (rural, urban, total) by state |
| `employment_st.csv` | LFPR, WPR, and unemployment (PU/1000) by state and gender |
| `mgnreg_st.csv` | MGNREG job cards, days worked, work sought-but-not-received per 1000 |
| `scholarships_st.csv` | Pre/post-matric scholarship amounts and utilization (2019-24) |
| `high_st_share.csv` | Flags the 19 high-ST-share states used in priority analysis |
| `tribal_villages.csv` | Count of villages by ST concentration threshold (50%, 75%, 90%, 100%) |
| `tribe_socioeconomic.csv` | Tribe-level literacy, WPR, and worker counts (aggregated) |
| `sc_st_residence.csv` | SC/ST population breakdown by rural/urban residence |
| `low_literacy_districts.csv` | Districts with female ST literacy below 35% |
| `household_type_rural.csv` | Rural household types by state |

---

## 4. Data Build Pipeline

**Entry point:** `scripts/build_project_data.py`

Running this script performs the entire data preparation workflow end to end:

### 4.1 Cleaning and Normalization

Each of the 17 source CSVs is loaded and cleaned through shared utility functions:

- **`normalize_state(name)`** — Harmonizes 30+ state name variants across datasets (e.g., `Orissa → Odisha`, `Uttaranchal → Uttarakhand`, `Jammu & Kashmir → Jammu and Kashmir`). This is the key function enabling cross-dataset merges.
- **`numeric(series)`** — Converts comma-separated strings, percentage suffixes, and mixed-type columns to float.
- **`clean_base(df, ...)`** — Applies normalization, strips aggregate rows (`India`, `All India`, `Total`, `All States`, etc.), and optionally extracts year columns.

Each dataset is cleaned into a standardized form and written to `outputs/cleaned/`.

### 4.2 State-Level Master Table

**`build_state_analysis_dataset()`** merges all 17 cleaned datasets on the normalized state name column using left joins, producing a single wide table with ~200 columns covering every metric for each of the 34 states/UTs.

Two versions are written:
- `outputs/analysis/state_analysis_dataset_all_states.csv` — all 34 states
- `outputs/analysis/state_analysis_dataset_high_st_states.csv` — the 19 high-ST-share states

### 4.3 Derived Scores

**`add_scores(df)`** computes three composite indicators on a 0–1 scale (higher = worse):

**Education Disadvantage Score** — weighted average of:
- ST literacy deprivation (100 minus literacy rate, scaled)
- Female ST literacy deprivation
- Literacy gap (ST vs total population)
- Secondary dropout rate
- Count of low female-literacy districts (normalized)

**Economic Vulnerability Score** — weighted average of:
- ST Below Poverty Line percentage
- Unemployment proxy (PU/1000 person-days)
- MGNREG unmet work demand (work sought but not received, per 1000)

**Overall Priority Score** — simple average of the two scores above. States are also assigned a `policy_priority_category` based on the combination of both scores (high/low on each dimension).

These scores drive the priority ranking table in `outputs/analysis/state_disadvantage_scores.csv`.

### 4.4 SQLite Export

**`write_sqlite()`** exports all 17 cleaned datasets plus the state analysis table into a single SQLite database at `outputs/st_education_project.sqlite`. This database powers both the in-dashboard SQL analyst and the standalone database analyzer app.

### 4.5 Pipeline Outputs Summary

| Output | Description |
|---|---|
| `outputs/cleaned/*.csv` | 17 normalized CSV files |
| `outputs/analysis/state_analysis_dataset_all_states.csv` | 34-state master table (~200 cols) |
| `outputs/analysis/state_analysis_dataset_high_st_states.csv` | 19 high-ST-state subset |
| `outputs/analysis/state_disadvantage_scores.csv` | Priority scores and rankings |
| `outputs/analysis/data_inventory.csv` | Metadata for all 17 datasets |
| `outputs/analysis/build_summary.md` | Build run statistics |
| `outputs/figures/` | Core summary plots (PNG) |
| `outputs/st_education_project.sqlite` | Complete SQLite database |

---

## 5. Analysis Outputs

### Master Dataset Columns (grouped by category)

**Demographic (6 columns)**
`st_population`, `total_population`, `st_share_state_population_pct`, `decadal_growth_pct`, rural/urban ST share

**Education — Literacy (5 columns)**
`st_literacy_rate_pct`, `total_literacy_rate_pct`, `literacy_gap_pct`, `st_female_literacy_pct`, `st_male_literacy_pct`

**Education — Enrolment (20+ columns)**
Enrolment counts by level (pre-primary through secondary), female share % at each level

**Education — GER (12 columns)**
GER for classes I–VIII, IX–X, XI–XII, by gender and year; latest 2023-24 GER with GPI variants

**Education — Dropout (3 columns)**
`dropout_primary_pct`, `dropout_upper_primary_pct`, `dropout_secondary_pct`

**Education — Scholarships (8 columns)**
Pre-matric and post-matric scholarship amounts, utilization %, amounts per 100k ST population

**Economic — Poverty (3 columns)**
`st_bpl_rural_pct`, `st_bpl_urban_pct`, `st_bpl_mean_pct`

**Economic — Employment (9 columns)**
LFPR, WPR, PU (unemployment proxy) per 1000 persons — disaggregated by gender

**Economic — MGNREG (10+ columns)**
Job cards per 1000, work provided per 1000, work days broken down by day-count bucket, average days worked, `mgnreg_sought_not_received_per_1000`

**Spatial — Tribal Villages (8 columns)**
Village counts at 50/75/90/100% ST concentration thresholds; normalized per 100k ST population

**Spatial — Low Literacy Districts (3 columns)**
`low_literacy_district_count`, `low_literacy_female_min_pct`, `low_literacy_female_mean_pct`

**Tribe Aggregates (5 columns)**
Tribe-level weighted literacy, WPR, tribe count, tribe total population

**Computed Scores (5 columns)**
`education_disadvantage_score`, `economic_vulnerability_score`, `overall_priority_score`, `overall_priority_rank`, `policy_priority_category`

---

## 6. Dashboard

**Entry point:** `dashboard_app/streamlit_app.py`
**Run with:** `streamlit run dashboard_app/streamlit_app.py`
**Access at:** `http://localhost:8501`

The dashboard is an interactive Streamlit application with a dark theme and a tabbed navigation structure. It loads `state_analysis_dataset_high_st_states.csv` (with caching) and renders all visualizations client-side using Plotly Express.

### 6.1 Overview Page

The landing view contains six interactive sections:

**KPI Cards**
Four summary metrics displayed as styled HTML cards at the top:
- Mean ST Literacy Rate (%)
- Mean Literacy Gap (%)
- Mean Secondary Dropout Rate (%)
- Mean MGNREG Unmet Demand (per 1000)

**Full India Choropleth Map**
A GeoJSON-backed choropleth of all 19 high-ST states. A dropdown lets the user select any column from the dataset as the fill metric. The map is rendered using Plotly's `choropleth_mapbox` with state boundary geometry from `india_state.geojson` (simplified with Shapely for fast rendering).

**Side-by-Side State Comparison Maps**
Two independent choropleth maps rendered in adjacent columns, each with its own metric dropdown, enabling direct geographic comparison of any two indicators.

**Interactive Scatter Plot Builder**
Dropdowns for X axis, Y axis, bubble size, and color metric. Renders an annotated scatter plot with a linear trend line (OLS) and Pearson correlation coefficient shown in the subtitle. State abbreviations are used as point labels.

**State Profile Sidebar**
A dropdown to select any state. Displays a table of all key metrics for that state alongside the 19-state median, with delta indicators (above/below median, color-coded by whether high is good or bad).

**Sortable State Comparison Table**
The full dataset as a paginated, sortable Streamlit dataframe with human-readable column labels. Users can sort by any column to rank states.

### 6.2 Question Tabs (Q1–Q13)

Thirteen research question tabs are exposed in the dashboard navigation. Each tab contains:
- A written question and context paragraph
- A Pearson correlation table (r and p-value for multiple related variable pairs)
- Two to four scatter plots with trend lines, color-coded by a relevant third variable
- Inline interpretation notes

| Tab | Research Focus |
|---|---|
| Q1 | ST literacy as a signal of long-term educational exclusion |
| Q2 | GER and enrolment vs dropout and poverty outcomes |
| Q3 | Enrolment sufficiency vs dropout as conflicting signals |
| Q4 | Literacy gap as a standalone indicator vs ST literacy alone |
| Q5 | Education-livelihood mismatch — decent schooling, poor livelihoods |
| Q6 | MGNREG unmet demand as a measure of livelihood distress |
| Q7 | Scholarship amounts and utilization vs dropout rates |
| Q8 | Schooling outcomes vs MGNREG dependency |
| Q9 | Gender parity in enrolment vs female literacy and workforce outcomes |
| Q10 | Secondary dropout as an early warning signal vs literacy |
| Q11 | Whether high-ST-share states systematically underperform |
| Q12 | Tribal village concentration and its relationship to outcomes |
| Q13 | Gender-specific disadvantages hidden inside state averages |

### 6.3 SQL Analyst Tab

An LLM-powered natural language query interface built into the dashboard. See [Section 7](#7-llm-engine) for a full description.

### 6.4 Styling and Technical Details

- **Theme:** Dark (#0d1117 background, #161b22 card background, #66b8e8 accent blue)
- **Charts:** Plotly Express with `plotly_dark` template, toolbar hidden, transparent backgrounds
- **Color scales:** `MAP_HEAT_SCALE` (yellow → orange → red) for risk metrics; `MAP_GOOD_SCALE` (same palette, inverted interpretation) for positive metrics; `CHART_COLOR_SCALE` for scatter/bar gradients
- **Caching:** All CSV loads use `@st.cache_data` to avoid reloading on widget interaction
- **Column labels:** A `LABELS` dict maps raw column names to human-readable display names throughout the UI
- **Risk direction:** `RISK_HIGH_COLUMNS` set tracks which columns are "higher = worse" for correct coloring of delta indicators

---

## 7. LLM Engine

The LLM engine is embedded in two places: the **SQL Analyst tab** in the main dashboard (`dashboard_app/streamlit_app.py`) and the **standalone database analyzer** (`database/app.py`). Both use the same underlying pattern.

### 7.1 Provider and Model

All LLM calls are routed through [OpenRouter](https://openrouter.ai), a unified API gateway for multiple LLM providers.

| Setting | Default | Environment Variable |
|---|---|---|
| API endpoint | `https://openrouter.ai/api/v1` | `LLM_BASE_URL` |
| Model (dashboard) | `openai/gpt-oss-120b:free` | `LLM_DEFAULT_MODEL` |
| Model (database app) | `tencent/hy3-preview:free` | `LLM_DEFAULT_MODEL` |
| API key | (required) | `OPENAI_API_KEY` |

The OpenAI Python client is initialized with the OpenRouter base URL, which means any OpenRouter-compatible model can be swapped in by changing the environment variable.

### 7.2 SQL Generation

**Function:** `generate_sql(question, schema_context, model)`

When the user types a natural-language question in the SQL Analyst tab, the LLM receives:

- **System prompt:** "You are a SQLite SQL expert. Return only valid SQLite SQL with no explanation or markdown fences. Use only the tables and columns listed in the schema."
- **User prompt:** The user's question, followed by the full database schema (table names, column names, and sample values for each table)

The LLM returns a raw SQL string. The function strips any residual markdown code fences before passing the query to the safety validator.

### 7.3 Safety Validation

**Function:** `is_safe_select_query(sql)`

Before any LLM-generated SQL is executed, it passes through a read-only safety check:

- Only `SELECT` and `WITH` statements are permitted
- Blocked keywords: `INSERT`, `UPDATE`, `DELETE`, `DROP`, `ALTER`, `TRUNCATE`, `CREATE`, `REPLACE`, `ATTACH`, `DETACH`
- A row limit (`LIMIT 200`) is enforced on all queries
- Any query failing validation is rejected with an error message displayed to the user — the database is never written to

### 7.4 Result Summarization

**Function:** `summarize_results(question, sql, result_df, model)`

An optional second LLM call that generates a natural-language summary of the query results. It receives:

- The original user question
- The SQL that was executed
- The first 20 rows of the result as CSV text

The model returns a 2–4 sentence narrative grounded in the actual data values, displayed below the result table in the dashboard.

### 7.5 Schema Context

The schema passed to the LLM is built dynamically at runtime by inspecting the live SQLite database: table names, all column names, and a few sample values per column. This ensures the LLM generates syntactically valid SQL against the real schema rather than hallucinating column names.

### 7.6 API Call Structure

Both SQL generation and summarization use the same call pattern:

```python
client = OpenAI(base_url=LLM_BASE_URL, api_key=OPENAI_API_KEY)

response = client.responses.create(
    model=model,
    input=[
        {"role": "system", "content": [{"type": "input_text", "text": system_prompt}]},
        {"role": "user",   "content": [{"type": "input_text", "text": user_prompt}]}
    ]
)

result_text = response.output_text
```

---

## 8. Database Analyzer App

**Entry point:** `database/app.py`
**Run with:** `streamlit run database/app.py`

A standalone Streamlit application that connects to a **MySQL** database (rather than SQLite) and exposes the same LLM-powered SQL analyst interface. Intended for use when the data is hosted in a shared MySQL server.

### Configuration

| Environment Variable | Description |
|---|---|
| `DATABASE_URL` | Full MySQL connection string (`mysql+pymysql://user:pass@host/db`) |
| `DB_SCHEMA` | Schema/database name (default: `dsm_final_project`) |
| `OPENAI_API_KEY` | OpenRouter API key |
| `LLM_DEFAULT_MODEL` | Model to use (default: `tencent/hy3-preview:free`) |
| `LLM_BASE_URL` | OpenRouter endpoint |

### Features

- **Schema browser:** Expandable sidebar showing all tables and columns
- **Natural language query box:** User types a question; the LLM generates SQL
- **Query display:** Shows the generated SQL before execution
- **Results table:** Paginated display of query results
- **LLM summary:** Toggle to enable/disable the narrative result summary
- **Error handling:** Failed queries show the SQL and error message for debugging

The SQL safety validator (`is_safe_select_query`) is applied identically to prevent any write operations.

---

## 9. EDA and Question Scripts

### `scripts/run_policy_eda.py`

A comprehensive exploratory data analysis script that generates an extended report in `outputs/eda/`. It produces:

- **Data quality tables** — coverage %, missing value counts, data source metadata
- **Summary statistics by priority tier** — means and distributions split by `policy_priority_category`
- **Pearson correlation matrices** — education vs economic variables, education vs MGNREG, etc.
- **PCA (Principal Component Analysis)** — dimensionality reduction on the 19-state dataset to identify the main axes of variation
- **K-Means clustering** — groups states into clusters based on combined education and economic indicators
- **Visualization outputs:** violin plots, heatmaps, pair plots, state-labelled scatter plots

All figures are saved as PNGs in `outputs/eda/figures/` and a written report is generated at `outputs/eda/eda_policy_report.md`.

### `scripts/generate_q*.py`

Six dedicated scripts generate publication-ready visualizations for specific research questions:

| Script | Research Focus | Output Directory |
|---|---|---|
| `generate_q2_graphs.py` | GER, enrolment, poverty, dropout relationships | `graphs/q2/` |
| `generate_q4_graphs.py` | Literacy gap informativeness analysis | `graphs/q4/` |
| `generate_q6_graphs.py` | MGNREG demand and livelihood distress | `graphs/q6/` |
| `generate_q8_graphs.py` | Schooling vs MGNREG dependency | `graphs/q8/` |
| `generate_q10_graphs.py` | Secondary dropout as a warning signal | `graphs/q10/` |
| `generate_q12_graphs.py` | Tribal village concentration outcomes | `graphs/q12/` |

Each script reads the cleaned analysis CSVs, applies Seaborn/Matplotlib styling, and saves annotated PNG charts.

---

## 10. Setup and Installation

### Prerequisites

- Python 3.9 or later
- An [OpenRouter](https://openrouter.ai) API key (free tier is sufficient for the default models)

### Install Dependencies

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install pandas numpy matplotlib scipy scikit-learn streamlit plotly shapely openai python-dotenv seaborn sqlalchemy pymysql
```

Or use the dashboard-specific requirements file:

```powershell
pip install -r dashboard_app/requirements.txt
```

### Environment Variables

Create a `.env` file in the project root (it is gitignored):

```env
# Required for LLM features
OPENAI_API_KEY=your_openrouter_api_key_here

# Optional overrides
LLM_BASE_URL=https://openrouter.ai/api/v1
LLM_DEFAULT_MODEL=openai/gpt-oss-120b:free

# Required only for the MySQL database app
DATABASE_URL=mysql+pymysql://user:password@host:3306/dsm_final_project
DB_SCHEMA=dsm_final_project
```

---

## 11. Running the Project

### Step 1 — Build the Data

Run the main pipeline from the repository root. This must be done before running the dashboard or EDA scripts.

```powershell
python scripts/build_project_data.py
```

Expected output: cleaned CSVs in `outputs/cleaned/`, master tables in `outputs/analysis/`, SQLite database at `outputs/st_education_project.sqlite`, and summary figures in `outputs/figures/`.

### Step 2 (Optional) — Run EDA

```powershell
python scripts/run_policy_eda.py
```

Generates extended EDA report and figures in `outputs/eda/`.

### Step 3 (Optional) — Generate Question Graphs

```powershell
python scripts/generate_q2_graphs.py
python scripts/generate_q4_graphs.py
python scripts/generate_q6_graphs.py
python scripts/generate_q8_graphs.py
python scripts/generate_q10_graphs.py
python scripts/generate_q12_graphs.py
```

### Step 4 — Launch the Dashboard

```powershell
streamlit run dashboard_app/streamlit_app.py
```

Open `http://localhost:8501` in your browser.

### Step 5 (Optional) — Launch the Database Analyzer

Requires a running MySQL instance with `DATABASE_URL` set in `.env`.

```powershell
streamlit run database/app.py
```

---

## 12. Research Questions

The project is structured around 13 empirical questions, each with dedicated visualizations in the dashboard and correlation analysis:

1. **Literacy as exclusion signal** — Does ST literacy rate predict employment and poverty outcomes?
2. **Schooling quality** — Do higher GER and enrolment translate to lower dropout and BPL rates?
3. **Dropout vs enrolment** — Do dropout rates reveal a different story than GER alone?
4. **Literacy gap informativeness** — Is the literacy gap (ST vs total) more predictive than ST literacy alone?
5. **Education-livelihood mismatch** — Which states have decent schooling metrics but still show poor livelihood outcomes?
6. **MGNREG as distress signal** — Does unmet MGNREG demand (work sought but not received) proxy for deeper economic vulnerability?
7. **Scholarship effectiveness** — Do states with higher per-capita scholarship release see lower dropout rates?
8. **Schooling vs MGNREG dependency** — Do states with better schooling outcomes still show high MGNREG dependency?
9. **Gender outcomes pipeline** — Does Gender Parity Index in enrolment translate to female literacy and WPR?
10. **Warning signal comparison** — Is secondary dropout a better early indicator than literacy rate for identifying at-risk states?
11. **High-ST-share performance** — Do states with larger ST population shares systematically underperform on outcomes?
12. **Spatial concentration effects** — Do states with more 100%-ST villages show worse or better outcomes?
13. **Gender-specific disadvantage** — Are female-specific disadvantages masked by state-level aggregates?

---

## 13. Priority State Rankings

Based on `outputs/analysis/state_disadvantage_scores.csv`, the current highest-priority high-ST states (combining education disadvantage and economic vulnerability scores) are:

1. Odisha
2. Madhya Pradesh
3. Jharkhand
4. Maharashtra
5. Rajasthan
6. Nagaland
7. Chhattisgarh
8. Jammu and Kashmir
9. Gujarat
10. Tripura

States ranked higher need more targeted policy intervention across both educational and livelihood dimensions. The full ranked table with individual score components is available in the dashboard's Overview page and in the `state_disadvantage_scores.csv` output file.

---

## Method Notes

- All state and UT names are normalized before cross-dataset merges to eliminate name variation artifacts.
- Aggregate rows (`India`, `All India`, `Total`) are removed from all state-level analysis.
- The 19 high-ST-share states are determined from the proposal dataset (`high_st_share.csv`).
- Original raw CSVs in `data/raw/` are never modified by any pipeline script.
- Composite scores use min-max normalization within the 19-state analysis set.
- Pearson correlation p-values are reported alongside r values in all correlation tables to indicate statistical significance.
