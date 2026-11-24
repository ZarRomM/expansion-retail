# Retail Store Expansion Recommender — Project Plan

Date: 2026-11-23 · Author: Mauricio Zárate Romero

## Summary

A recommender that ranks the Mexican municipalities where a retail format should open its next store. It is proven by comparing its recommendations with openings that actually happened.

- **Problem:** deciding where to open the next store.
- **Data:** INEGI's business directory (DENUE) plus population and income data by municipality.
- **Test:** what share of next year's real openings were in the model's ranking.
- **Deliverable:** a published interactive map and a README that can be read in 2 minutes.

## The business problem

A chain's expansion team decides where each new store goes. A bad location means a lost investment or a store that closes later.

**Project question (draft, to be refined):** in which municipalities is a given store format most likely to open a store next year?

| Format | Example chains |
| --- | --- |
| Supermarket | Walmart, Soriana |
| Hard discount | 3B, Neto |
| Wholesale / cash & carry | El Zorro and similar |
| Convenience | OXXO, 7-Eleven, Circle K |

Each format gets its own model, because each one picks locations with different logic.

The format is assigned by chain, not by INEGI's industry classification, because that classification does not separate hard discount from supermarkets.

**Out of scope:** sales forecasting. Public data has no sales figures for private chains.

## The data

The main source is DENUE (National Statistical Directory of Economic Units): a list of businesses with name, location and activity. Comparing its historical snapshots shows which stores appear (openings) and which disappear (closures).

| Source | What it provides | Used for |
| --- | --- | --- |
| DENUE historical snapshots (INEGI) | Businesses with name, location and activity | Counting stores by chain, municipality and period; detecting openings and closures |
| Population and Housing Census (INEGI) | Population and dwellings by municipality | Market size |
| Municipal poverty indicators (CONEVAL) | Socioeconomic level | Which format fits each municipality |
| Geostatistical framework (INEGI) | Municipal boundaries | Drawing the map |

**Why the data is messy:** the same chain is spelled many ways ("TIENDAS 3B", "3 B", "BBB"), under different legal names, and with duplicates. Cleaning this is a core part of the project.

To confirm at the start: which DENUE snapshots are available for download, and how often they are published.

## How it is built

The project follows an ELT approach: raw data is stored as it arrives and transformed afterwards with SQL. If a cleaning rule changes, it is re-run without downloading anything again.

1. **Extract.** Download the DENUE snapshots and the other sources. Store them unchanged.
2. **Load.** Load the raw data into PostgreSQL.
3. **Transform.** With SQL:
    - Standardize each chain's name and assign its format.
    - Remove duplicates.
    - Flag a store as an opening when it appears in one snapshot and not in the previous one.
    - Aggregate by municipality and period.
4. **Features.** Per municipality: population, socioeconomic level, own and competitor stores, recent openings, and whether neighboring municipalities already have a store.
5. **Model.** For each municipality and period, estimate the probability that the format opens at least one store in the next period.
    - First, a simple baseline: rank municipalities by population.
    - Then, Machine Learning models (logistic regression and gradient boosting). They only count if they beat the baseline.
6. **Ranking.** Sort municipalities from highest to lowest probability.

Tools: Python, SQL, PostgreSQL and Git. Processed data is stored as Parquet and experiments are tracked with MLflow, for tracking only. Nothing else unless needed.

## Validation against reality

This is what sets the project apart: it proves the model gets real openings right.

1. Train the model only on data up to year T.
2. Produce the municipality ranking for year T+1.
3. Compare it with the real openings in T+1 from DENUE.

**Main metric:** share of real openings that fell in the top 50 ranked municipalities (recall@50).

**Key rule:** the model must beat the population baseline. If it does not, it adds no value, and that is reported too.

**Watch for data leakage:** no feature may use data from after year T.

**Result sentence (template):** "Of the stores [chain] opened in [year], X% were in my top 50 recommended municipalities, vs. Y% for the population baseline."

## Interactive map

A simple web app, published at a link, that a recruiter can open without installing anything.

- Format selector: supermarket, hard discount, wholesale or convenience.
- Municipality map colored by opening probability.
- Table of the top 50 recommended municipalities.
- Validation view: where the model was right and where it missed.

Proposed tool: Streamlit, published for free on Streamlit Community Cloud. Built last, once the model and validation are done.

## README and definition of done

The repository README must be understandable in 2 minutes, with four parts in this order:

1. **The problem:** one or two sentences.
2. **The data:** where it comes from and what was cleaned.
3. **The validation:** how it was tested against real openings.
4. **The result:** three lines, with the result sentence and the link to the map.

The project is done when:

- [ ] The pipeline runs end to end from raw data.
- [ ] Openings by chain and format are identified and hand-checked on a sample.
- [ ] The model was compared against the population baseline.
- [ ] Validation on year T+1 is done and reported, good or bad.
- [ ] The map is published and opens from a link.
- [ ] The README has all four parts.

## Open decisions

To be resolved before writing code, in this order:

- [ ] Final project question in one sentence: what is predicted, at what level, and for which decision.
- [ ] Exact list of chains in each format.
- [ ] Which DENUE snapshots are used and which year T is held out for validation.
- [ ] Operational definition of an opening and a closure.
- [ ] Geographic level: municipality, unless the data forces another one.
- [ ] Convenience format: decide whether the target is "opens at least one store" or "how many it opens", since OXXO is already in almost every urban municipality.
