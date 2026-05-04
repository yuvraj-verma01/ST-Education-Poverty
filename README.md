# DSM Final Project: ST Education and Livelihood Analysis

This repository contains the data cleaning pipeline, analysis outputs, visualizations, and Streamlit dashboard for a DSM final project on Scheduled Tribe (ST) education and livelihood outcomes across Indian states and union territories.

The project combines education, demographic, employment, poverty, MGNREG, scholarship, and tribal-village datasets into reproducible state-level analysis tables. The main analytical focus is identifying high-ST-share states with overlapping education disadvantage and economic vulnerability.

## Repository Structure

```text
data/raw/                 Short, reproducible copies of the raw source CSVs
scripts/                  Data build, EDA, and question-specific graph scripts
outputs/cleaned/          Cleaned normalized CSV datasets
outputs/analysis/         Combined state tables, scores, correlations, and build summary
outputs/eda/              EDA report tables and figures
outputs/figures/          Core summary figures from the build pipeline
graphs/q*/                Question-specific chart outputs
dashboard_app/            Streamlit dashboard and dashboard README
database/                 SQL/database-related project files
```

Large original source files with long filenames are kept at the root or under `Additional(Potential)Data/`, while the pipeline primarily reads the shorter tracked copies under `data/raw/`.

## Main Outputs

Key generated files include:

- `outputs/analysis/state_analysis_dataset_all_states.csv`
- `outputs/analysis/state_analysis_dataset_high_st_states.csv`
- `outputs/analysis/state_disadvantage_scores.csv`
- `outputs/analysis/correlations.csv`
- `outputs/analysis/build_summary.md`
- `outputs/st_education_project.sqlite`
- `outputs/eda/eda_policy_report.md`
- `graphs/q*/`

The build summary reports 17 cleaned datasets, 34 state-level rows in the combined table, and 19 high-ST-share states.

## Setup

Create and activate a virtual environment, then install the Python dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install pandas numpy matplotlib scipy scikit-learn streamlit plotly shapely openai python-dotenv
```

The dashboard-specific dependency list is also available at `dashboard_app/requirements.txt`.

## Rebuild the Data

Run the main project data build from the repository root:

```powershell
python scripts/build_project_data.py
```

This regenerates cleaned CSVs, analysis tables, the SQLite database, and core figures.

To regenerate the broader EDA outputs:

```powershell
python scripts/run_policy_eda.py
```

Question-specific graphs can be regenerated with the scripts in `scripts/`, for example:

```powershell
python scripts/generate_q2_graphs.py
python scripts/generate_q4_graphs.py
python scripts/generate_q6_graphs.py
python scripts/generate_q8_graphs.py
python scripts/generate_q10_graphs.py
python scripts/generate_q12_graphs.py
```

## Run the Dashboard

From the project root:

```powershell
streamlit run dashboard_app/streamlit_app.py
```

The dashboard reads the generated analysis outputs and provides KPI cards, state comparisons, sortable priority tables, relationship plots, and question tabs for the project visuals.

## Method Notes

- State and UT names are normalized before merging datasets.
- Aggregate rows such as India, All India, and Total are removed from state-level analysis.
- High-ST-share states are identified from the proposal dataset.
- The combined analysis table includes education disadvantage, economic vulnerability, and overall priority scores.
- Original raw CSVs are not edited by the build pipeline.

## Current Priority Snapshot

Based on `outputs/analysis/build_summary.md`, the highest-scoring high-ST states in the generated priority table include Odisha, Madhya Pradesh, Jharkhand, Maharashtra, Rajasthan, Nagaland, Chhattisgarh, Jammu and Kashmir, Gujarat, and Tripura.
