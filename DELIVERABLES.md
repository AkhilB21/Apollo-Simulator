# Deliverables — Apollo Build File Inventory

**Machine-auditable inventory of every file in this build.**
An external AI auditor can verify: (a) every listed file exists, (b) no unlisted files exist, (c) each file's stated purpose matches its actual content.

**Build Version**: <!-- SET BY BUILDER -->
**Build Date**: <!-- SET BY BUILDER -->
**Zip Filename**: <!-- SET BY BUILDER -->
**Total Files**: <!-- SET BY BUILDER: count after zip -->

---

## Audit Instructions

1. List all files in the extracted zip: `find . -type f | sort`
2. Compare against the tables below.
3. Every file in the zip MUST appear in these tables.
4. Every file in these tables MUST exist in the zip.
5. `__pycache__/` directories are exempt from listing.
6. Flag any discrepancies.

---

## Standard Build Files (MUST be in every build)

| File | Purpose | Changed This Version? |
|------|---------|---------------------|
| `DEFINITION_OF_DONE.md` | Build quality checklist — must be filled in by builder, verified by auditor | YES (new) |
| `DELIVERABLES.md` | This file — machine-auditable file inventory | YES (new) |
| `PROJECT_INSTRUCTIONS.md` | Master build instructions — read before every zip | YES |
| `CHANGELOG.md` | Version history — must have entry for current version | YES |
| `README.md` | User-facing documentation — must reference current version | YES |
| `requirements.txt` | Python dependencies | Maybe |
| `run_apollo.bat` | Launch backtest Streamlit dashboard (CRLF line endings) | Maybe |
| `run_apollo_silent.vbs` | Silent Streamlit launcher (no CMD window) | No |
| `run_sync_data.bat` | Sync CSV source data to Parquet repository (CRLF line endings) | Maybe |
| `run_audit.bat` | Launch data audit tool — zero-arg, auto-generates HTML report (CRLF line endings) | Maybe |
| `apollo_data_audit.py` | Data quality auditor — validates CSV + Parquet integrity | Maybe |
| `build_universe.bat` | Build universe JSON from Parquet data | No |
| `build_universe.py` | Universe builder script | No |
| `apollo_universe.json` | Symbol universe (auto-generated) | No |
| `ipo_listing_dates.json` | IPO listing dates for IPO-aware detection | No |
| `smallcap500.json` | Small-cap 500 stock list | No |
| `etmoney_stocks_list.json` | Etmoney screener stock list | No |
| `my_watchlist.json` | User's personal watchlist | No |
| `tests/preflight.py` | Pre-flight test suite — must pass before zip | Maybe |

---

## apollo_core/ — Shared Scoring Engine

| File | Purpose | Dependencies | Changed This Version? |
|------|---------|--------------|---------------------|
| `__init__.py` | Package marker | — | No |
| `config.py` | Single source of truth for APOLLO_DATA_ROOT path, utility functions | — | YES (created) |
| `constants.py` | Score thresholds, pool definitions, exit parameters | — | Maybe |
| `scoring.py` | 21 mature signals + 12 IPO signals (the scoring brain) | indicators, types, constants | Maybe |
| `indicators.py` | RSI, MACD, ADX, Williams %C, TruePrice computation | numpy, pandas | Maybe |
| `trade_engine.py` | run_backtest, detect_exit_triggers, execute_exit, extract_trades | scoring, types, indicators | Maybe |
| `types.py` | Position, ExitConfig, ExitDecision dataclasses | dataclasses | Maybe |
| `bucket_classifier.py` | Layer-1 stock classification (REFERENCE ONLY — no gating) | constants | No |
| `renko.py` | Renko brick computation | numpy | No |
| `gates.py` | Hard entry gates (PE, liquidity, 52W proximity) | fundamentals | No |
| `fundamentals.py` | PE and fundamental data lookups | pandas | No |
| `ipo_lookup.py` | Listing-date-aware IPO detection | ipo_listing_dates.json | No |
| `regime_prefilter.py` | Market regime detection (Bull/Bear/Sideways) | yfinance, numpy | No |

---

## data_repo/ — Central Data Repository Module

| File | Purpose | Dependencies | Changed This Version? |
|------|---------|--------------|---------------------|
| `__init__.py` | Package marker — exports load_daily, save_daily, list_symbols | repo | YES |
| `repo.py` | Parquet read/write — load_daily, save_daily, list_symbols, merge_bars, manifest | pandas, pyarrow, json | YES |
| `sources.py` | CSV loaders — load_eod2_csv with flexible date parsing | pandas | No |
| `sync.py` | CLI to sync all CSVs into Parquet repository | sources, repo, pandas | YES |

---

## backtest_engine/ — Streamlit Backtest Dashboard

| File | Purpose | Dependencies | Changed This Version? |
|------|---------|--------------|---------------------|
| `__init__.py` | Package marker | — | No |
| `app.py` | Streamlit dashboard — main UI, version string in title | streamlit, plotly, apollo_core, guidance_engine, data_repo | Maybe |
| `backtest.py` | High-level backtest wrapper per symbol | trade_engine, eod2_loader | No |
| `eod2_loader.py` | Data loader — reads from data_repo Parquet, computes indicators | data_repo, apollo_core.indicators | YES |
| `backtest_history.py` | SQLite history DB — save/load/query backtest results | sqlite3, json, apollo_core.config | No |
| `report_generator.py` | Generate XLSX/HTML backtest reports | openpyxl, plotly | No |

---

## guidance_engine/ — Post-Trade Analysis Layer

| File | Purpose | Dependencies | Changed This Version? |
|------|---------|--------------|---------------------|
| `__init__.py` | Package marker — exports run_guidance_pipeline | aggregator, analyzer, recorder | No |
| `schemas.py` | Data classes + all guidance constants (single source of truth) | dataclasses | No |
| `recorder.py` | Record per-trade MAE/MFE data to trades.parquet | schemas, apollo_core.config | No |
| `mae_mfe.py` | Compute MAE/MFE curves and summary statistics | pandas, numpy | No |
| `analyzer.py` | Per-stock behavioral profiles, pattern detection | recorder, schemas, mae_mfe | No |
| `regime.py` | NIFTY 50 + India VIX regime detection and caching | yfinance, apollo_core.config | No |
| `flag_generator.py` | Generate behavioral flags per trade | analyzer, schemas | No |
| `aggregator.py` | Systemic aggregation — bucket-level flag frequency | flag_generator, schemas, recorder | No |
| `validate_data.py` | Validate trades.parquet schema before pipeline | recorder | No |

---

## live_engine/ — Live Monitoring + Dashboard

| File | Purpose | Dependencies | Changed This Version? |
|------|---------|--------------|---------------------|
| `__init__.py` | Package marker | — | No |
| `__main__.py` | Enables `python -m live_engine` | cli | No |
| `cli.py` | Command-line interface for live monitor | argparse, signal_monitor | No |
| `signal_monitor.py` | SignalMonitor — bar-by-bar signal detection | trade_engine, state_store, data_replay | No |
| `data_replay.py` | BarReplay — feeds CSV/Parquet data bar-by-bar | pandas | No |
| `state_store.py` | SQLite state persistence + crash recovery | sqlite3 | No |
| `dashboard.py` | Streamlit dashboard for live monitoring | streamlit, plotly | No |
| `run_live.py` | Single launcher for the Streamlit dashboard | dashboard | No |
| `alert_manager.py` | File log + Telegram alert delivery | — | No |
| `daily_report.py` | Generate daily monitoring reports | — | No |
| `watchlist.py` | Watchlist management | — | No |
| `multi_monitor.py` | Multi-symbol parallel monitoring | signal_monitor | No |
| `run_headless.py` | Headless mode (no UI) | cli | No |

---

## nse_engine/ — NSE Data Pipeline

| File | Purpose | Dependencies | Changed This Version? |
|------|---------|--------------|---------------------|
| `__init__.py` | Package marker | — | No |
| `__main__.py` | Enables `python -m nse_engine` | run_pipeline | No |
| `run_pipeline.py` | Main NSE data pipeline orchestrator | scanner, data_feed | No |
| `scanner.py` | Stock scanner with screening criteria | yfinance, pandas | No |
| `data_feed.py` | Data feed from NSE sources | yfinance, requests | No |

---

## Batch Files — Windows Launchers

| File | Purpose | Notes |
|------|---------|-------|
| `run_nse_engine.bat` | Launch NSE pipeline | CRLF required |
| `run_apollo_live.bat` | Launch live engine dashboard | CRLF required |
| `sync_to_gdrive.bat` | Sync APOLLO_DATA_REPOSITORY to Google Drive | CRLF required |

---

## Scripts (root level)

| File | Purpose | Dependencies | Notes |
|------|---------|--------------|-------|
| `sync_to_gdrive.py` | rclone-based Google Drive sync | rclone (external) | One-time rclone config needed |
