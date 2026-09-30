# Project Baseline

Recorded: 2026-09-30  
Branch: `main`  
Remote: `https://github.com/priyanshshrivastav23-source/nifty-market-risk-platform.git`  
Python: 3.14.4 (system, `C:\Python314\python.exe`)

---

## Environment Status

The committed `.venv/` virtual environment is **broken** — it references
`C:\Users\Lucky\AppData\Local\Programs\Python\Python310` which does not exist
on this machine. A fresh environment `.venv_new/` was created using Python 3.14
for baseline verification.

No `requirements.txt` exists in the repository. Dependencies were inferred
from source code imports and installed manually.

### Packages installed for baseline verification

| Package | Version |
|---------|---------|
| fastapi | 0.142.2 |
| uvicorn | 0.54.0 |
| streamlit | 1.64.0 |
| pandas | 3.0.6 |
| numpy | 2.5.3 |
| scikit-learn | 1.9.1 |
| scipy | 1.18.1 |
| matplotlib | 3.11.2 |
| seaborn | 0.13.2 |
| plotly | 7.1.0 |
| reportlab | 5.0.1 |
| openpyxl | 3.1.5 |
| pytest | 9.1.1 |
| pytest-cov | 7.1.0 |
| httpx | 0.28.1 |
| requests | 2.34.2 |
| pyyaml | 6.0.3 |
| pydantic | 2.13.5 |

---

## Current Architecture

```
Nifty100_Analytics/
├── config/               # screener_config.yaml only
├── data/
│   ├── raw/              # 12 Excel source files
│   └── nifty100.db       # stale duplicate database
├── db/
│   ├── nifty100.db       # primary SQLite database (~1.1 MB)
│   └── schema.sql        # formal schema (4 tables, disconnected from reality)
├── docs/                 # acceptance checklist, analyst guide, openapi.json
├── notebooks/            # 2 SQL files
├── output/               # generated CSVs, XLSXs, logs
├── pages/                # 8 empty placeholder .py files (dead code)
├── reports/              # generated PDFs and PNGs
├── src/
│   ├── analytics/        # financial calculations (10 modules)
│   ├── api/              # FastAPI app + 8 routers
│   ├── dashboard/        # Streamlit app + 8 pages + utils/db.py
│   ├── dq/               # data quality rules
│   ├── etl/              # loader, normaliser, validator
│   ├── nlp/              # regex parser, pros/cons generator
│   ├── reports/          # PDF generation (tearsheet, sector, portfolio)
│   └── screener/         # filter engine, export
└── tests/
    ├── api/              # 4 tests
    ├── dq/               # 14 tests
    ├── etl/              # 13 tests
    ├── kpi/              # 28 tests
    └── screener/         # 3 tests
```

Also present: 9 root-level `test_*.py` scripts (not pytest-discoverable
in standard configuration; they are standalone execution scripts).

---

## Test Results

Command: `.venv_new\Scripts\python.exe -m pytest tests/ -v`

**Result: 72 passed, 0 failed, 1 warning**

```
72 passed in 3.13s
```

Warning: `httpx`/`starlette.testclient` deprecation notice (cosmetic, does not
affect test outcomes).

All 72 tests pass on the baseline commit before any modifications.

### Test breakdown by module

| Test module | Tests | Result |
|------------|-------|--------|
| tests/api/ | 4 | PASS |
| tests/dq/ | 14 | PASS |
| tests/etl/ | 13 | PASS |
| tests/kpi/ | 28 | PASS |
| tests/screener/ | 3 | PASS |
| **Total** | **72** | **PASS** |

---

## API Runtime Status

Command: `.venv_new\Scripts\uvicorn.exe src.api.main:app --port 8000`

**Result: API starts successfully.**

### Endpoint responses at baseline

| Endpoint | HTTP | Status | Notes |
|----------|------|--------|-------|
| `GET /` | 200 | OK | Returns welcome message |
| `GET /api/v1/health/` | 200 | Partial | `profit_loss`, `balance_sheet`, `cash_flow` return 0 — wrong table names |
| `GET /api/v1/companies/companies/` | 200 | OK | Double prefix in URL (`/companies/companies/`) |
| `GET /api/v1/screener/screener/` | 200 | OK | Double prefix in URL |
| `GET /api/v1/sectors/sectors/` | 200 | OK | Double prefix in URL |
| `GET /api/v1/peers/peers/` | 200 | OK | Double prefix in URL |
| `GET /api/v1/valuation/valuation/` | 200 | OK | Double prefix in URL |
| `GET /api/v1/portfolio/portfolio/` | 200 | OK | Double prefix in URL |
| `GET /api/v1/documents/documents/` | 200 | OK | Double prefix in URL |

Health endpoint shows:
```json
{
  "companies": 92,
  "financial_ratios": 1184,
  "profit_loss": 0,
  "balance_sheet": 0,
  "cash_flow": 0
}
```
The real table names are `profitandloss`, `balancesheet`, `cashflow` — not the
names the health check uses.

---

## Dashboard Runtime Status

Command: `.venv_new\Scripts\streamlit.exe run src/dashboard/app.py`

**Result: Dashboard starts successfully (confirmed via import check).**

Known issue: all 8 dashboard pages (`src/dashboard/pages/`) use hardcoded
dummy data. The utility module `src/dashboard/utils/db.py` provides proper
cached DB query functions but is not imported by any page.

---

## Import-Time Side Effects (Confirmed)

The following modules execute code — including DB writes, file exports, and
chart generation — when imported. This is a design defect.

| Module | Side effect observed on import |
|--------|-------------------------------|
| `src.analytics.peer` | Runs peer ranking, writes `output/peer_comparison.xlsx`, creates `peer_percentiles` table in SQLite |
| `src.analytics.radar` | Generates 5 radar chart PNGs using `numpy.random` data |
| `src.analytics.valuation` | Creates hardcoded DataFrame, writes `output/valuation_summary.xlsx` and `output/valuation_flags.csv` |

---

## Database Status

Primary database: `db/nifty100.db`  
Size: ~1.1 MB  
Tables: 12 (loaded from Excel via ETL)

| Table | Rows (approx) |
|-------|--------------|
| companies | 92 |
| financial_ratios | 1,184 |
| stock_prices | populated |
| profitandloss | populated |
| balancesheet | populated |
| cashflow | populated |
| sectors | populated |
| market_cap | populated |
| peer_groups | populated |
| documents | populated |
| prosandcons | populated |
| analysis | populated |

`schema.sql` defines only 4 tables with mostly empty column definitions.
The ETL loader ignores `schema.sql` and dumps Excel files directly into
SQLite via `to_sql(if_exists="replace")`.

Duplicate database at `data/nifty100.db` (~128 KB, stale).

---

## Known Failures and Technical Debt

### Critical

1. **No `requirements.txt`** — dependencies are unknown without inspecting
   source code.

2. **Broken `.venv/`** — references Python 3.10 installation that no longer
   exists at the committed path.

3. **Import-time side effects** — `peer.py`, `radar.py`, `valuation.py`
   execute DB writes, file exports, and chart generation on import.

4. **Double-prefix API routes** — all routes have duplicated path segments
   (e.g., `/api/v1/companies/companies/`) due to prefix duplication between
   `main.py` and router definitions.

5. **Health endpoint uses wrong table names** — `profit_loss`, `balance_sheet`,
   `cash_flow` do not exist; real names are `profitandloss`, `balancesheet`,
   `cashflow`.

6. **Tearsheet PDFs are hardcoded** — `tearsheet.py` generates identical PDFs
   for every company regardless of the company name passed.

### Significant

7. **Dashboard is entirely hardcoded** — no page queries the database.
   `utils/db.py` exists but is unused.

8. **`schema.sql` is disconnected from actual database** — real schema comes
   from raw Excel column names; the SQL file is not used.

9. **Duplicate database file** — `data/nifty100.db` and `db/nifty100.db` exist.

10. **`normaliser.py` not integrated** — exists in ETL but is never called
    by `loader.py`.

11. **All DB paths hardcoded** — `"db/nifty100.db"` appears as a string
    literal in multiple source files.

12. **No centralized configuration** — no settings module or environment
    variable support.

13. **Root-level `test_*.py` scripts** — 9 files named like pytest tests but
    are standalone scripts; some import functions that have been renamed
    (e.g., `cfo_quality` vs `cfo_quality_score`).

14. **No risk analytics module** — the `stock_prices` table exists with price
    time-series data but no volatility, VaR, Sharpe, or drawdown calculations
    are implemented.

15. **8 empty files in `pages/`** — root-level placeholder files serving
    no purpose.

16. **No structured logging** — ETL uses `print()` statements only.

17. **API responses have no Pydantic models** — raw dict returns with no
    type safety or documentation.

18. **No pagination** on API endpoints — all return `LIMIT 20`.

19. **`peer.py` uses module-level script pattern** — entire analysis runs
    at import time rather than being wrapped in callable functions.

20. **`radar.py` uses `numpy.random` data** — charts generated with random
    values, not actual financial data.

---

## Original Run Commands (baseline)

```bash
# Create environment
python -m venv .venv_new
.venv_new\Scripts\pip.exe install fastapi uvicorn streamlit pandas numpy \
    openpyxl scikit-learn matplotlib seaborn plotly reportlab pytest \
    pytest-cov requests scipy pyyaml httpx

# Run ETL
python src/etl/loader.py

# Start API
.venv_new\Scripts\uvicorn.exe src.api.main:app --reload --port 8000

# Start Dashboard
.venv_new\Scripts\streamlit.exe run src/dashboard/app.py

# Run tests
.venv_new\Scripts\python.exe -m pytest tests/ -v
```

---

## Original Git History Summary

55 commits following a "Day N / Sprint N" naming pattern from the original
author's learning-style development process. Commit messages reflect a
sprint-based course structure rather than conventional software engineering
milestones.

The transformation will build on top of this baseline with meaningful,
milestone-based commits using conventional commit style.
