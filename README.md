# S&P 500 Portfolio Performance & Risk Analysis

End-to-end data engineering and quantitative analysis pipeline that builds a point-in-time Top 10 portfolio from S&P 500 constituents, evaluates its out-of-sample performance (2021 onwards), and compares risk-adjusted metrics against the SPY benchmark.

---

## Project Overview

This project demonstrates a complete workflow covering:

- **Data extraction** – scraping the current S&P 500 constituent list from Wikipedia
- **Data modelling** – storing company metadata and historical OHLCV prices in a SQLite star schema
- **Transformation** – calculating daily adjusted returns
- **Portfolio construction** – ranking stocks on 2016–2020 total return (point-in-time) and selecting the Top 10
- **Out-of-sample evaluation** – measuring performance and risk from 2021 to present
- **Benchmarking & visualisation** – comparing the portfolio against SPY and charting drawdowns

The goal is to showcase data engineering, SQL, Python analytics, and basic quantitative finance skills while applying a clean methodological framework that avoids look-ahead bias in stock selection.

---

## Key Results (as of last run)

### Top 10 Stocks (ranked on 2016–2020 total return)

| Rank | Ticker | Company                  | Sector                     | Total Return 2016–2020 |
|------|--------|--------------------------|----------------------------|------------------------|
| 1    | AMD    | Advanced Micro Devices   | Information Technology     | 3210.83%               |
| 2    | XYZ    | Block, Inc.              | Financials                 | 1689.80%               |
| 3    | NVDA   | Nvidia                   | Information Technology     | 1548.76%               |
| 4    | TSLA   | Tesla, Inc.              | Consumer Discretionary     | 1479.32%               |
| 5    | VEEV   | Veeva Systems            | Health Care                | 848.94%                |
| 6    | ALGN   | Align Technology         | Health Care                | 731.46%                |
| 7    | GNRC   | Generac                  | Industrials                | 686.61%                |
| 8    | AXON   | Axon Enterprise          | Industrials                | 628.05%                |
| 9    | IDXX   | Idexx Laboratories       | Health Care                | 602.85%                |
| 10   | PODD   | Insulet Corporation      | Health Care                | 589.77%                |

### Risk & Performance Metrics (Test Period: 2021 onwards)

| Metric                | Top 10 Portfolio | SPY    |
|-----------------------|------------------|--------|
| Ann. Return (%)       | 18.08            | 14.80  |
| Ann. Volatility (%)   | 32.67            | 16.80  |
| Sharpe Ratio          | 0.49             | 0.76   |
| Sortino Ratio         | 0.74             | 1.05   |
| Max Drawdown (%)      | -55.71           | -24.50 |

The Top 10 portfolio delivered higher absolute returns but with substantially higher volatility and deeper drawdowns, resulting in weaker risk-adjusted performance versus SPY.

---

## Step-by-Step Pipeline

### Step 0 – Project Setup
- Install required packages (`yfinance`, `pandas`, `numpy`, `sqlalchemy`, `matplotlib`, `seaborn`, etc.)
- Import libraries and configure logging for pipeline transparency

### Step 1 – Database Schema & Architecture
- Create a SQLite engine (`sp500_db_engine.db`)
- Define a star-schema DDL:
  - `dim_assets` – ticker, company name, sector, industry
  - `fact_daily_prices` – OHLCV + adjusted close (primary key: ticker + date)
  - `fact_daily_returns` – daily percentage returns

### Step 2 – Metadata Extraction & Ingestion
- Scrape current S&P 500 constituents from Wikipedia (BeautifulSoup + pandas)
- Standardise tickers (e.g. `BRK.B` → `BRK-B`) for yfinance compatibility
- Append SPY as the benchmark instrument
- Load cleaned metadata into `dim_assets` (clear-and-reload pattern for safe re-runs)

### Step 3 – Historical Price Extraction
- Retrieve all tickers from `dim_assets`
- Bulk-download daily OHLCV data from 2016 onwards via `yfinance`
- Reshape the MultiIndex output into a flat relational format
- Map columns to the `fact_daily_prices` schema and load into SQLite

### Step 4 – Daily Returns Calculation
- Query adjusted close prices ordered by ticker and date
- Compute percentage change (`pct_change`) within each ticker group
- Drop the first NaN row per ticker and insert results into `fact_daily_returns`

### Step 5 – Point-in-Time Top 10 Portfolio & Risk Analysis
- **Ranking window (2016–2020):** SQL window functions calculate total return using first and last available adjusted close; filter for stocks that traded near the start of the period; select Top 10
- **Test window (2021–present):** Pull daily returns for the Top 10 + SPY
- Construct an equal-weighted portfolio with dynamic rebalancing (NaNs skipped)
- Calculate risk metrics: Annualised Return, Volatility, Sharpe, Sortino, Maximum Drawdown
- Generate a side-by-side comparison table and drawdown chart versus SPY

### Why SPY instead of ^GSPC for the benchmark?
Individual stock returns use **adjusted close** prices (dividends + splits included). SPY also reflects total return, providing a fair like-for-like comparison. The pure price index ^GSPC would understate the benchmark.

---

## Important Limitations

1. **Survivorship bias** – Analysis uses the *current* S&P 500 membership. Companies delisted or removed from the index between 2016–2026 are excluded, which tends to inflate historical returns.
2. **Partial look-ahead bias** – While ranking is strictly point-in-time, the investable universe is today’s constituents rather than the actual index membership that existed in 2016–2020.
3. **Portfolio construction** – Equal weighting with dynamic rebalancing (NaNs skipped). This is not a pure buy-and-hold strategy.

Results should be interpreted as an **exploratory analysis**, not a production-ready investable backtest.

---

## Tech Stack

| Category          | Tools                                      |
|-------------------|--------------------------------------------|
| Language          | Python 3                                   |
| Data Extraction   | `requests`, BeautifulSoup, `yfinance`      |
| Data Storage      | SQLite + SQLAlchemy                        |
| Analysis          | pandas, NumPy                              |
| Visualisation     | matplotlib, seaborn                        |
| Environment       | Jupyter Notebook                           |

---

## Repository Structure

```
sp500-performance-risk-analysis/
├── README.md
├── requirements.txt
├── .gitignore
└── notebooks/
    └── S&P_500_Performance_Risk_Analysis.ipynb
```

> **Note:** The SQLite database (`sp500_db_engine.db`) is generated at runtime and intentionally excluded from version control.

---

## Getting Started

### Prerequisites
- Python 3.9+
- Jupyter Notebook or JupyterLab

### Installation

```bash
git clone https://github.com/<your-username>/sp500-performance-risk-analysis.git
cd sp500-performance-risk-analysis

python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

### Run the Analysis

```bash
jupyter notebook notebooks/S&P_500_Performance_Risk_Analysis.ipynb
```

Execute the cells in order. The notebook will:
1. Create the SQLite database and schema
2. Scrape constituents and download price history
3. Rank the Top 10 and compute out-of-sample metrics
4. Display tables and drawdown charts

> **Runtime note:** The initial `yfinance` bulk download for ~500 tickers can take several minutes depending on network conditions.

---

## Requirements

```
yfinance
pandas
numpy
sqlalchemy
matplotlib
seaborn
beautifulsoup4
requests
lxml
```

A ready-to-use `requirements.txt` is included in the repository.

---

## Future Improvements

- Point-in-time S&P 500 membership (to fully eliminate survivorship bias)
- Alternative ranking signals (momentum, volatility-adjusted returns, fundamental screens)
- Transaction cost modelling and realistic rebalancing schedules
- Multi-factor portfolio construction
- Interactive dashboard (Streamlit / Dash)
