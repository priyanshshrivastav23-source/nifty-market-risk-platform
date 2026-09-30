# NIFTY Market Data & Risk Analytics Platform

A portfolio-grade financial data engineering and analytics platform for NIFTY 100
stocks. The platform ingests structured financial data from Excel sources, stores
it in a relational SQLite database, exposes analytics through a FastAPI REST
service, and presents results through an interactive Streamlit dashboard.

---

## Overview

This project covers the full analytics stack for Indian equity markets:

- **Data pipeline** — structured ETL from 12 Excel source files into SQLite
- **Financial analytics** — profitability, leverage, efficiency, and growth ratios
- **Cash flow intelligence** — FCF, CFO quality, CapEx intensity, distress signals
- **Stock screener** — six preset investment strategies with composite scoring
- **Peer comparison** — within-group percentile ranking across financial metrics
- **Clustering** — KMeans segmentation of stocks by financial profile
- **REST API** — FastAPI service with eight domain routers
- **Interactive dashboard** — Streamlit multi-page application

---

## Architecture

```
data/raw/*.xlsx          12 Excel source files
       │
       ▼
src/etl/loader.py        Extraction, column deduplication, null handling
       │
       ▼
db/nifty100.db           SQLite — 12 tables
       │
       ├──► src/analytics/     Financial ratio calculations
       │         ratios.py     NPM, ROE, ROCE, ROA, D/E, ICR, asset turnover
       │         cagr.py       Compound annual growth rate (6 edge cases)
       │         cashflow_kpis.py  FCF, CFO quality, CapEx intensity, distress
       │         clustering.py KMeans segmentation (5 clusters)
       │         peer.py       Within-group percentile rankings
       │         screener/     Filter engine + composite scoring + Excel export
       │
       ├──► src/api/           FastAPI — 8 routers
       │
       └──► src/dashboard/     Streamlit — 8 pages
```

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Language | Python 3.10+ |
| Database | SQLite (via `sqlite3`) |
| Data processing | Pandas, NumPy |
| Machine learning | scikit-learn (KMeans, StandardScaler) |
| Statistics | SciPy |
| API | FastAPI, Uvicorn |
| API validation | Pydantic |
| Dashboard | Streamlit |
| Visualization | Plotly, Matplotlib, Seaborn |
| PDF reports | ReportLab |
| Excel export | openpyxl |
| Testing | pytest, pytest-cov |
| Configuration | python-dotenv |

---

## Project Structure

```
.
├── config/
│   └── screener_config.yaml      Screener threshold configuration
├── data/
│   └── raw/                      12 Excel source files
├── db/
│   ├── nifty100.db               SQLite database (~1.1 MB, 92 companies)
│   └── schema.sql                Reference schema
├── docs/                         Technical documentation
├── output/                       Generated CSVs and Excel reports
├── reports/                      Generated PDFs and charts
├── src/
│   ├── analytics/
│   │   ├── cagr.py               CAGR with edge-case handling
│   │   ├── capital_report.py     Capital allocation pattern analysis
│   │   ├── cashflow_kpis.py      Cash flow intelligence functions
│   │   ├── cluster_profile.py    Cluster statistics and outlier detection
│   │   ├── clustering.py         KMeans stock segmentation
│   │   ├── peer.py               Peer group percentile ranking
│   │   ├── radar.py              Radar chart generation
│   │   ├── ratios.py             Financial ratio calculations
│   │   └── valuation.py          FCF yield and valuation flag functions
│   ├── api/
│   │   ├── database.py           SQLite connection factory
│   │   ├── main.py               FastAPI application and middleware
│   │   └── routers/              8 API routers
│   ├── config/
│   │   └── settings.py           Centralized configuration
│   ├── dashboard/
│   │   ├── app.py                Streamlit entry point
│   │   ├── pages/                8 dashboard pages
│   │   └── utils/db.py           Cached database query functions
│   ├── dq/
│   │   └── rules.py              14 data quality validation rules
│   ├── etl/
│   │   ├── loader.py             Excel → SQLite pipeline
│   │   ├── normaliser.py         Year, ticker, and text normalization
│   │   └── validator.py          Pre-load data quality checks
│   ├── nlp/
│   │   ├── parser.py             CAGR text parser (regex)
│   │   └── pros_cons_generator.py Rule-based investment signal generator
│   ├── reports/
│   │   ├── portfolio_summary.py  Portfolio PDF generation
│   │   ├── sector_report.py      Sector PDF generation
│   │   └── tearsheet.py          Company tearsheet PDF
│   └── screener/
│       ├── engine.py             Filter engine and preset strategies
│       └── export.py             Composite scoring and Excel export
└── tests/
    ├── api/                      API integration tests (4 tests)
    ├── dq/                       Data quality tests (14 tests)
    ├── etl/                      ETL unit tests (13 tests)
    ├── kpi/                      Financial calculation tests (28 tests)
    └── screener/                 Screener tests (3 tests)
```

---

## Database

The SQLite database at `db/nifty100.db` contains **92 NIFTY 100 companies**
across **12 tables** loaded from Excel source files.

| Table | Description | Key Columns |
|-------|-------------|-------------|
| `companies` | Company master | ticker, company_name, sector |
| `profitandloss` | P&L history per company per year | revenue, net_profit, operating_profit |
| `balancesheet` | Balance sheet history | equity, borrowings, total_assets |
| `cashflow` | Cash flow history | cfo, capex, investing, financing |
| `financial_ratios` | Pre-computed ratios | roe_percentage, roce_percentage, net_profit_margin_pct |
| `stock_prices` | Historical price data | date, open, high, low, close, volume |
| `sectors` | Sector classification | broad_sector, sub_sector |
| `peer_groups` | Peer group assignments | peer_group_name, company_id |
| `market_cap` | Market capitalisation | market_cap_cr |
| `analysis` | CAGR text fields | compounded_sales_growth, stock_price_cagr |
| `documents` | Annual report links | document_url, year |
| `prosandcons` | Rule-based signals | pros, cons |

> The ETL loader uses `to_sql(if_exists="replace")` — each run fully reloads
> all tables from the source Excel files.

---

## Financial Analytics

All financial calculations are implemented as pure, side-effect-free functions.
Each function returns `None` (rather than raising) for mathematically undefined
inputs such as zero denominators.

### Profitability
| Function | Formula |
|----------|---------|
| `net_profit_margin(net_profit, sales)` | (Net Profit / Sales) × 100 |
| `operating_profit_margin(op_profit, sales)` | (EBIT / Sales) × 100 |
| `return_on_equity(net_profit, equity, reserves)` | (Net Profit / Equity) × 100 |
| `return_on_capital_employed(ebit, equity, reserves, borrowings)` | (EBIT / Capital Employed) × 100 |
| `return_on_assets(net_profit, total_assets)` | (Net Profit / Total Assets) × 100 |

### Leverage & Efficiency
| Function | Notes |
|----------|-------|
| `debt_to_equity(borrowings, equity, reserves)` | Returns 0 for debt-free companies |
| `high_leverage_flag(de_ratio, sector)` | Threshold > 5; exempt for Financials sector |
| `interest_coverage(op_profit, other_income, interest)` | Returns None for zero-interest (debt-free) |
| `interest_warning_flag(icr)` | True when ICR < 1.5 |
| `asset_turnover(sales, total_assets)` | Sales / Total Assets |
| `net_debt(borrowings, investments)` | Borrowings − Investments |

### CAGR
`calculate_cagr(start, end, years)` returns `(value, flag)` where flag is one of:

| Flag | Condition |
|------|-----------|
| `NORMAL` | Standard positive growth |
| `ZERO_BASE` | Starting value is zero |
| `TURNAROUND` | Negative start, positive end |
| `DECLINE_TO_LOSS` | Positive start, negative end |
| `BOTH_NEGATIVE` | Both values negative |
| `INVALID_PERIOD` | Years ≤ 0 |

### Cash Flow Intelligence
| Function | Description |
|----------|-------------|
| `free_cash_flow(cfo, capex)` | CFO + CapEx (CapEx is typically negative) |
| `cfo_quality_score(cfo, net_profit)` | CFO/PAT ratio → "High Quality" / "Low Quality" |
| `capex_intensity(investing, sales)` | CapEx as % of Sales |
| `fcf_conversion_rate(fcf, cfo)` | FCF / CFO × 100 |
| `capital_allocation_pattern(cfo, investing, financing)` | Classifies as Reinvestor / Mature / Aggressive / Other |
| `distress_signal(cfo, financing)` | True when CFO negative + heavy borrowing |
| `deleveraging(financing, old_borrowings, new_borrowings)` | True when debt is being reduced |

---

## Stock Screener

The screener in `src/screener/engine.py` filters a DataFrame of companies
by financial metrics. Six preset investment strategies are built in:

| Strategy | Key Criteria |
|----------|-------------|
| **Quality Compounder** | ROE ≥ 18, D/E ≤ 1, Revenue CAGR ≥ 12, FCF > 0 |
| **Value Pick** | P/E ≤ 20, P/B ≤ 2, ROE ≥ 12 |
| **Growth Accelerator** | Revenue CAGR ≥ 20, PAT CAGR ≥ 20 |
| **Dividend Champion** | Dividend Yield ≥ 2, Payout ≤ 80%, Sales ≥ 5000 Cr |
| **Debt-Free Blue Chip** | D/E = 0, ROE ≥ 15, FCF > 0 |
| **Turnaround Watch** | PAT CAGR > 0, Revenue CAGR > 0, FCF > 0 |

#### Composite Quality Score

Each screened company is assigned a composite score (0–100):

```
Score = (ROE × 0.35) + (FCF × 0.30) + (Revenue CAGR × 0.20) + ((100 − D/E) × 0.15)
```

Export writes an Excel file with conditional formatting: green for top quartile,
red for bottom quartile.

---

## API Endpoints

Start the API:

```bash
uvicorn src.api.main:app --reload --port 8000
```

Interactive docs: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/` | Welcome and version |
| `GET` | `/api/v1/health/` | Service health and DB row counts |
| `GET` | `/api/v1/companies/companies/` | List companies (limit 20) |
| `GET` | `/api/v1/screener/screener/` | Screener — filter by name or ROE |
| `GET` | `/api/v1/sectors/sectors/` | Sector breakdown with company counts |
| `GET` | `/api/v1/peers/peers/` | Peer group assignments |
| `GET` | `/api/v1/valuation/valuation/` | Financial ratios |
| `GET` | `/api/v1/portfolio/portfolio/` | Portfolio analytics view |
| `GET` | `/api/v1/documents/documents/` | Annual report links |

Example — health check:
```bash
curl http://127.0.0.1:8000/api/v1/health/
```
```json
{
  "status": "ok",
  "version": "1.0.0",
  "uptime_seconds": 42,
  "db_row_counts": {
    "companies": 92,
    "financial_ratios": 1184,
    "profitandloss": 736,
    "balancesheet": 736,
    "cashflow": 736
  }
}
```

---

## Dashboard

Start the dashboard:

```bash
streamlit run src/dashboard/app.py
```

Opens at [http://localhost:8501](http://localhost:8501)

| Page | Description |
|------|-------------|
| Home | Market overview, KPI summary, sector distribution |
| Company Profile | Per-company KPIs, revenue trend, ROE trend, pros/cons |
| Stock Screener | Preset strategy selector and financial metric filters |
| Peer Comparison | Radar chart and metric table for peer groups |
| Financial Trends | Multi-year revenue and profit growth charts |
| Sector Analysis | Sector-level bubble chart (ROE vs Revenue) |
| Capital Allocation | Treemap of capital allocation patterns |
| Annual Reports | Document viewer and report links |

---

## Data Quality

`src/dq/rules.py` validates DataFrames against 14 business rules before
or after loading. Rules check for:

- Negative sales, profit, or debt values
- Zero or invalid equity and revenue
- ROE exceeding 100% (likely a data error)
- Missing company names, sectors, years, or cashflow fields
- Duplicate tickers
- Invalid market cap or share count values

---

## Testing

```bash
# Run all tests
pytest tests/ -v

# Run with coverage report
pytest tests/ --cov=src --cov-report=term-missing

# Run a specific suite
pytest tests/kpi/ -v
```

**Baseline result: 72 tests, 0 failures**

| Suite | Tests | Coverage area |
|-------|-------|--------------|
| `tests/api/` | 4 | HTTP status codes, response shapes |
| `tests/dq/` | 14 | All 14 data quality rules |
| `tests/etl/` | 13 | Column deduplication, year/ticker normalization, validation |
| `tests/kpi/` | 28 | CAGR edge cases, all ratio functions, cashflow KPIs, screener presets |
| `tests/screener/` | 3 | Filter engine, composite score, Excel export |

---

## Setup and Installation

**Requirements:** Python 3.10 or later, Git

```bash
# 1. Clone the repository
git clone https://github.com/priyanshshrivastav23-source/nifty-market-risk-platform.git
cd nifty-market-risk-platform

# 2. Create and activate a virtual environment
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. (Optional) Configure environment variables
cp .env.example .env
# Edit .env to override defaults — all settings have safe defaults

# 5. Run the ETL pipeline to populate the database
python src/etl/loader.py

# 6. Start the API
uvicorn src.api.main:app --reload --port 8000

# 7. Start the dashboard (separate terminal)
streamlit run src/dashboard/app.py

# 8. Run tests
pytest tests/ -v
```

---

## Configuration

All configurable settings live in `src/config/settings.py`. Override any
value by setting the corresponding environment variable or by creating a
`.env` file in the project root (see `.env.example`).

| Variable | Default | Description |
|----------|---------|-------------|
| `APP_ENV` | `development` | Application environment |
| `LOG_LEVEL` | `INFO` | Logging verbosity |
| `DB_PATH` | `db/nifty100.db` | Path to SQLite database |
| `OUTPUT_DIR` | `output` | Directory for generated reports |
| `REPORTS_DIR` | `reports` | Directory for PDF and chart output |
| `DATA_DIR` | `data/raw` | Directory containing source Excel files |
| `API_HOST` | `127.0.0.1` | API bind address |
| `API_PORT` | `8000` | API port |
| `RISK_FREE_RATE` | `0.065` | Risk-free rate for Sharpe/Sortino (annualised) |
| `VAR_CONFIDENCE` | `0.95` | Confidence level for Value at Risk |

---

## Design Decisions

**SQLite over PostgreSQL** — The dataset is 92 companies with historical data.
SQLite is sufficient for the analytics workload and removes a deployment
dependency. The connection pattern is compatible with PostgreSQL if scaling
is required.

**Pure functions for financial calculations** — All ratio and KPI functions
in `src/analytics/` are pure: no database access, no file I/O, no global
state. This makes them directly testable and reusable across the API and
dashboard layers.

**Screener operates on DataFrames** — The screener engine accepts a Pandas
DataFrame rather than querying the database directly. This decouples the
filtering logic from the data access layer and allows it to be used in
isolation in tests without a live database.

**KMeans clustering on 5 features** — Stocks are segmented using five
financial dimensions: ROE, D/E, Revenue CAGR, FCF CAGR, and operating margin.
The elbow method selects the optimal cluster count. Clusters are assigned
human-readable labels (e.g., "High Quality Compounders", "Emerging Growth").

---

## Known Limitations

- The ETL loader performs a full replace on every run — there is no
  incremental load or change detection.
- Dashboard pages currently display pre-loaded data; real-time filtering
  against the live database is under development.
- API endpoints return up to 20 records with no pagination parameters.
- The `schema.sql` file is a reference document; the actual database schema
  is derived from the Excel source file column headers.

---

## License

This project is built on top of an open-source codebase. The source
repository was used as a starting point and has been substantially
redesigned, extended, and refactored. Original source material is licensed
under its original terms. All original engineering additions in this
repository — including the configuration layer, risk analytics module,
ETL redesign, and API refactoring — are the work of this project's author.
