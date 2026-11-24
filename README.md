# Retail Store Expansion Recommender (Mexico)

> Work in progress. Full plan: [docs/plan.md](docs/plan.md).

Which Mexican municipalities should a retail chain open its next store in? This project ranks municipalities by the probability that a store format opens a new location there next year, and tests those rankings against store openings that actually happened.

## The problem

A chain's expansion team decides where each new store goes. A bad location means a lost investment or a store that closes later. This project supports that decision with public data, separately for four store formats:

| Format | Example chains |
| --- | --- |
| Supermarket | Walmart, Soriana |
| Hard discount | 3B, Neto |
| Wholesale / cash & carry | El Zorro and similar |
| Convenience | OXXO, 7-Eleven, Circle K |

## The data

- **DENUE (INEGI):** Mexico's national business directory. Comparing its historical snapshots shows which stores opened and which closed.
- **Population and Housing Census (INEGI):** market size by municipality.
- **CONEVAL poverty indicators:** socioeconomic level by municipality.
- **INEGI geostatistical framework:** municipal boundaries for the map.

The raw data is messy: the same chain appears under many spellings and legal names, with duplicates. Cleaning and matching it is a core part of the pipeline.

## The validation

The model is trained only on data up to year T, ranks municipalities for year T+1, and is scored against the real openings in T+1.

- **Metric:** share of real openings that fall in the top 50 recommended municipalities (recall@50).
- **Baseline:** ranking municipalities by population. The model only counts as useful if it beats it.

## The result

<!-- Three lines: the result sentence, the baseline comparison, and the link to the map. -->
<!-- Template: "Of the stores [chain] opened in [year], X% were in my top 50 municipalities, vs. Y% for the population baseline." -->

## Interactive map

<!-- Link to the published Streamlit app. -->

## Project structure

```
data/           raw, interim and processed data (not tracked in Git)
sql/            transformations: staging → intermediate → marts
src/expansion/  extraction, features, models, evaluation
notebooks/      exploration only
app/            Streamlit map
docs/           plan, data dictionary, decision records
```

## How to run

<!-- Steps to reproduce the pipeline from raw data. -->

## Tech stack

Python, SQL, PostgreSQL, Parquet, MLflow (experiment tracking), Streamlit.
