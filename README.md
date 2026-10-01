# Build Guide: Regional Demand Forecasting & Expansion Prioritisation

**Goal:** One public GitHub repo + one public Tableau dashboard, finished in **14 days**, that reads as a business solution to UK/US employers.

**The pitch (use this everywhere):** *"I built a forecasting and prioritisation system that tells a multi-region retailer where demand is heading and which regions to invest in next, using official Stats NZ data. It cuts forecast error by X% vs a naive baseline, worth an estimated $Y in avoided stockouts and overstock."*

**Skills this proves:** Python, time-series forecasting, feature engineering, validation, unsupervised learning, hypothesis testing, deployment, business communication.

## Ground rules

1. **Ship ugly, then polish.** A finished simple project beats an unfinished brilliant one.
2. **Simple models first.** Show that you tried complex ones, but recommend whichever wins on validation.
3. **If the API blocks you for more than half a day, download CSVs from Aotearoa Data Explorer** and move on. Add the API script later.
4. **Commit to GitHub every day.** A commit history shows real work.

---

## Day 1: Setup and framing

- [ ] Create a public GitHub repo: `nz-regional-demand-forecasting`.
- [ ] Set up Python 3.11+, a virtual environment, and install: `pandas numpy scikit-learn statsmodels lightgbm matplotlib seaborn requests jupyter pytest`.
- [ ] Create this structure:

```
nz-regional-demand-forecasting/
├── README.md
├── requirements.txt
├── data/raw/  data/processed/
├── notebooks/ (01_eda, 02_features, 03_models, 04_segmentation, 05_stats_tests)
├── src/ (ingest.py, features.py, models.py, evaluate.py)
├── outputs/ (CSV files for Tableau)
└── tests/
```

- [ ] Write the **problem statement** in the README now (5 lines): the company (fictional building-supplies retailer with branches in NZ regions), the pain (overstock/stockouts, unclear where to expand), the decision (stock levels and next branch location), the stakeholder (Head of Operations).
- [ ] Register at **portal.apis.stats.govt.nz**, subscribe, and create your subscription key. Store it in a `.env` file and add `.env` to `.gitignore`. **Never commit your key.**

## Days 2-3: Get the data

- [ ] In **explore.data.stats.govt.nz**, find and note the dataset IDs for these, by region and by month or quarter where possible:
  - Building consents (leading indicator for building-supply demand)
  - Retail trade or card spending
  - Population estimates and migration
  - Labour market (employment/unemployment by region)
  - CPI (national, for price context)
- [ ] Follow the Stats NZ API user guide and use its code generator (Python) to pull each dataset. Write `src/ingest.py` so one command downloads everything to `data/raw/`.
- [ ] Clean into one tidy table: `region, date, indicator, value`. Handle missing values and changed region names. Save to `data/processed/`.
- [ ] Document each source (name, URL, licence, date pulled) in the README.
- **Check:** at least 5 years of monthly or quarterly data for at least 8 regions. If you can't get regional retail data, use regional building consents as your **demand proxy** and say so openly.

## Days 4-5: Exploration and feature engineering

- [ ] Plot every series. Note seasonality, trend, the COVID shock, and outliers.
- [ ] Define your **target** (for example, regional building consents or retail value) and justify it as a demand proxy.
- [ ] Build features in `src/features.py`:
  - Lags (1, 3, 6, 12 periods) and rolling means/std
  - Calendar features (month, quarter)
  - Leading indicators from other series (lagged consents, migration, employment)
  - A COVID-period flag
- [ ] **No leakage:** features may only use information available at forecast time. Write one test in `tests/` that checks this.

## Days 6-8: Forecasting models

- [ ] **Baselines:** seasonal naive and last-year-same-period. Everything must beat these.
- [ ] **Statistical:** SARIMA or ETS per region (`statsmodels`).
- [ ] **ML:** LightGBM or Random Forest on the engineered features, one global model across regions.
- [ ] **Optional deep model:** a small LSTM or Transformer-style forecaster. Include it only if you have time, and be honest if it loses. Saying "the simpler model won, so I recommend it" is a strength.
- [ ] **Validation:** walk-forward (expanding window) cross-validation, never random splits. Report MAE, RMSE and MAPE per region and overall.
- [ ] Produce a comparison table and a plot of forecast vs actual with prediction intervals.
- [ ] **Business value:** assume illustrative costs (for example, a stockout costs $A per unit of unmet demand, overstock costs $B). Convert forecast error into dollars for your best model vs the naive baseline. State these assumptions clearly as illustrative.
- [ ] Save final forecasts to `outputs/forecasts.csv`.

## Day 9: Segmentation and expansion score

- [ ] Cluster regions using demographics and economic features (standardise first, then K-means or hierarchical; justify K with silhouette score).
- [ ] Name the clusters in business language ("High-growth urban", "Stable regional", and so on).
- [ ] Build a transparent **expansion score** per region: weighted combination of forecast growth, market size, and volatility (risk). Keep weights simple and explain them.
- [ ] Save to `outputs/segments.csv` and `outputs/expansion_ranking.csv`.

## Day 10: Hypothesis testing

- [ ] Pick one real business question, such as: *"Did demand in region X structurally shift after \[event\]?"* or *"Do high-growth clusters differ significantly in demand volatility?"*
- [ ] Use an appropriate test (t-test or Mann-Whitney, ANOVA or Kruskal-Wallis, or an interrupted time series), check assumptions, report effect size and a confidence interval, not just a p-value.
- [ ] Write the conclusion in plain English for a manager.
- [ ] *Optional experiment design:* add a short power analysis and a simulated A/B test plan for a pricing or promotion change. Label it as simulated.

## Days 11-12: Tableau Public dashboard

- [ ] Download Tableau Public (free) and sign up. Connect to your CSVs in `outputs/`.
- [ ] Build **one dashboard for a manager**, with:
  1. NZ regional map coloured by expansion score or forecast growth
  2. Forecast vs actual line chart with a region filter
  3. Ranked bar chart of regions for investment
  4. KPI tiles: forecast error improvement and estimated annual value
  5. A short "Recommended actions" text box (3 bullets)
- [ ] Use a clean palette, clear titles that state the insight ("Waikato is forecast to grow fastest"), and tooltips.
- [ ] Publish to Tableau Public. Test the link in a private browser window.
- [ ] Note that data is a **snapshot** and refreshes only when you re-publish.

## Day 13: README and polish

README order matters. Recruiters may read only the top.

1. Title, one-line pitch, **dashboard link**, screenshot
2. Business problem and the decision it supports
3. Key results (3 numbers: error reduction, estimated value, top-ranked regions)
4. Recommendations
5. Approach (data, features, models, validation) in a short diagram or list
6. How to reproduce (install, add API key, run commands)
7. Assumptions and limitations (including illustrative costs and the demand proxy)
8. Next steps (for example, add UK ONS data to show the method transfers)

- [ ] Clean notebooks (restart and run all), remove dead code, add `requirements.txt`, pin versions.
- [ ] Add 2-3 passing tests.

## Day 14: Go to market

- [ ] **CV bullet (adapt with your real numbers):** *"Built a regional demand forecasting and expansion-ranking system on Stats NZ API data; walk-forward validated LightGBM/SARIMA models reduced MAPE by X% vs seasonal baseline (est. $Y annual value); delivered via a public Tableau dashboard."*
- [ ] Put the GitHub and Tableau links at the top of your CV, LinkedIn Featured section, and email signature.
- [ ] Post a short LinkedIn write-up: problem, result, one chart, links.
- [ ] Start applying immediately, then iterate. Don't wait for perfection.

---

## Applying to UK and US employers

- **Visas:** check which employers sponsor (UK Skilled Worker / Graduate routes, US H-1B). Filter job searches for sponsorship and apply there first.
- **Tailor the pitch:** for retail, supply chain, property, and consulting roles, lead with the business framing. For tech roles, lead with the pipeline and validation.
- **Transferability line:** *"The same pipeline works with ONS (UK) or Census/FRED (US) data."* Even better, add one UK or US series later as a v2.
- **Fill the gaps:** computer vision and NLP still need small separate projects. Start them after this one is live, and apply while you build them.

## Common pitfalls

- Random train/test splits on time series (invalid, use walk-forward)
- Data leakage from future-looking features
- Using a deep model when a simple one wins, without justification
- Dashboard with too many charts and no recommendation
- Committing your API key
- Claiming real savings without stating your assumptions

## Daily checklist

- [ ] Code committed and pushed
- [ ] Notes added to the README draft
- [ ] One thing finished, not five things started