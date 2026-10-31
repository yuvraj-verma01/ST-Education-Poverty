# ST Education and Livelihood Dashboard

## Run

From the project root:

```powershell
streamlit run dashboard_app/streamlit_app.py
```

The app reads the existing project outputs:

- `outputs/analysis/state_analysis_dataset_high_st_states.csv`
- `outputs/analysis/state_analysis_dataset_all_states.csv`
- `outputs/analysis/state_disadvantage_scores.csv`

## What It Shows

- Project-level KPI cards
- State explorer and sortable priority table
- Interactive relationship explorer
- Question tabs for Q1-Q13 with the most important visuals from the notebook logic
- Correlation values under each interactive plot

## Hosted dashboard

The browser-based dashboard is hosted directly on GitHub Pages:

https://yuvraj-verma01.github.io/dsmfinalproj.github.io/dashboard/

It includes maps, state comparisons, all 13 research views, downloads, and a
read-only browser SQL explorer. Its source and datasets are versioned in
[yuvraj-verma01/dsmfinalproj.github.io](https://github.com/yuvraj-verma01/dsmfinalproj.github.io/tree/main/dashboard).
The AI analyst uses a separate server API with a private provider credential;
its availability is reported in the dashboard. Charts and manual SQL do not
require that API. The Streamlit source in this directory is preserved.
