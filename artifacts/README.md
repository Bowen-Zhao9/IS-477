# COVID-19 / Pharmaceutical Stock Analysis Pipeline

A reproducible Snakemake workflow that investigates the relationship between
U.S. COVID-19 severity and the daily performance of three pharmaceutical
stocks (CVS, JNJ, ABBV) over 2021.

## Project structure

```
.
├── Snakefile                       # Workflow definition (DAG)
├── run_all.sh                      # Snakemake-free fallback runner
├── requirements.txt                # Python dependencies
├── config/
│   └── config.yaml                 # All tunable parameters
├── scripts/
│   ├── 01_load_covid.py            # Concatenate JHU CSSE CSVs
│   ├── 02_clean_covid.py           # Daily diffs + national aggregation
│   ├── 03_fetch_stocks.py          # Per-ticker yfinance download
│   ├── 04_integrate.py             # Next-trading-day merge + clip
│   ├── 05_feature_engineering.py   # Rolling, Z-score, momentum, severity
│   ├── 06_analyze.py               # Per-ticker OLS regressions
│   └── 07_visualize.py             # Multi-panel price-vs-cases plot
├── csse_covid_19_daily_reports_us/ # Input data (NOT generated)
├── data/processed/                 # Intermediate CSVs (generated)
├── results/
│   ├── analysis/                   # Regression summaries (generated)
│   └── figures/                    # Plots (generated)
└── logs/                           # Per-rule run logs (generated)
```

## Workflow DAG

```
csse CSVs ─▶ load_covid ─▶ clean_covid ──┐
                                         ├─▶ integrate ─▶ features ─┬─▶ analyze
yfinance ─▶ fetch_stock × N ─────────────┘                          └─▶ visualize
```

## How to run

### With Snakemake (recommended)

```bash
pip install -r requirements.txt
snakemake --cores 1            # full pipeline
snakemake -np                  # dry-run preview
snakemake --forceall --cores 1 # rebuild everything
snakemake clean                # wipe generated artifacts
```

### Without Snakemake

```bash
pip install -r requirements.txt
bash run_all.sh
```

## Configuration

All tunable parameters live in `config/config.yaml`:

- `tickers` — list of stock symbols to analyze
- `start_date` / `end_date` — range passed to yfinance
- `year_filter` — substring used to select COVID CSVs
- `infection_window` / `mortality_window` — rolling-mean windows
- `infection_weight` / `mortality_weight` — composite-index weights

Editing the YAML and re-running `snakemake` re-executes only the downstream
rules whose inputs changed.

## Outputs

| File                                       | Description                                |
|--------------------------------------------|--------------------------------------------|
| `data/processed/covid_raw_combined.csv`    | All daily JHU rows for the chosen year     |
| `data/processed/covid_state_daily.csv`     | Cleaned state-level series with daily diffs|
| `data/processed/covid_us_daily.csv`        | National daily New_Confirmed / New_Deaths  |
| `data/processed/stock_<TICKER>.csv`        | Per-ticker Close / Volume                  |
| `data/processed/integrated.csv`            | COVID + stocks aligned to trading days     |
| `data/processed/features.csv`              | Adds rolling, Z, momentum, severity, returns|
| `results/analysis/ols_<TICKER>.txt`        | Full statsmodels OLS summary               |
| `results/analysis/regression_results.csv`  | Tidy table of all coefficients across stocks|
| `results/figures/stocks_vs_cases.png`      | Multi-panel price-vs-cases visualization   |

## Data source

Place the JHU CSSE daily-report CSVs in
`csse_covid_19_daily_reports_us/` (one per day, named `MM-DD-YYYY.csv`).
They can be obtained from
<https://github.com/CSSEGISandData/COVID-19/tree/master/csse_covid_19_data/csse_covid_19_daily_reports_us>.

Stock prices are pulled from the public Yahoo Finance API by `yfinance`
and require an internet connection at runtime.
