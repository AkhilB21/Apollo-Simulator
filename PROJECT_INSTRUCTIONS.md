# APOLLO — Project Instructions

Single source of truth for all engine deliveries. Read this BEFORE every zip is built.

---

## 1. File Naming Convention (added 2026-07-27)

**Format:** `[ENGINE_NAME]_[ddmmyy]_v[N].zip`

| Component | Rule | Example |
|---|---|---|
| `ENGINE_NAME` | Engine identifier | `Apollo_Backtest`, `Apollo_Live` |
| `ddmmyy` | Day, month, year — 2 digits each | `270726` (27 July 2026) |
| `vN` | Auto-incremented per iteration, restarting at v1 each new day | `v1`, `v2`, `v3` |

**Examples:**
- `Apollo_Backtest_270726_v1.zip` — first backtest-engine delivery on 27 July 2026
- `Apollo_Backtest_270726_v2.zip` — second iteration same day (bugfix)
- `Apollo_Live_280726_v1.zip` — first live-engine delivery on 28 July 2026

**Triggered by:** Any request that changes engine code.

**Additional rules:**
- Old zips from prior days are kept (history). Old zips from the SAME day are deleted when a newer version is delivered (no clutter).
- Individual changed files (e.g., just `app.py`) follow the same pattern: `app_270726_v2.py`
- Always include the versioned zip + individual changed files in `/home/z/my-project/download/`

---

## 2. Title-Version Sync (from Conversation for Next Version.txt L2526)

The landing-page title line in `backtest_engine/app.py` must always read:

> `Multi-timeframe RSI Recovery Scoring Engine — NSE Backtester vX.Y.Z — <codename>`

The version in the title MUST match the zip's version. No stale version strings allowed.

There is an HTML comment in `app.py` (around line 272) reminding of this rule. The pre-flight Test 12 enforces it programmatically.

---

## 2b. Version-Bump Checklist (mandatory since v4.3)

Every time engine code changes (new feature, bugfix, signal change), ALL of the following must be updated before zipping:

| # | What | Where | Example for v4.3 |
|---|------|-------|-------------------|
| 1 | Module header docstring | `apollo_core/scoring.py`, `constants.py`, `trade_engine.py` line 1 | `"""... (v4.3)"""` |
| 2 | App.py file header | `backtest_engine/app.py` line 10 | `Version: 4.3 — ...` |
| 3 | App.py HTML title | `backtest_engine/app.py` ~line 270 | `NSE Backtester v4.3 — Renko Hard Gate` |
| 4 | Pre-flight constants | `tests/preflight.py` lines 25-26 | `CURRENT_VERSION = "v4.3"` |
| 5 | Stale version list | `tests/preflight.py` ~line 210 | Add old version to `stale_versions` |
| 6 | CHANGELOG.md | Top of file | New `## Apollo Backtest vX.Y — ...` section |
| 7 | README.md | Line 1 | `# Apollo Engine vX.Y — ...` |
| 8 | PROJECT_INSTRUCTIONS.md | Change Log table | Row with date + summary |
| 9 | Zip filename | `/home/z/my-project/download/` | `Apollo_Live_300726_v1.zip` |

---

## 2c. Mandatory Build Files (enforced since v4.12.1+)

Every Apollo zip MUST include these two files, filled in by the builder:

| File | Purpose | When Updated |
|------|---------|-------------|
| `DEFINITION_OF_DONE.md` | Machine-auditable quality checklist — builder fills header + checks, auditor verifies | Every build |
| `DELIVERABLES.md` | Complete file inventory with purpose, dependencies, change status — auditor cross-checks against actual zip contents | Every build |

### Build Pipeline (mandatory order)

1. **Write/modify code** for the feature or fix.
2. **Update all version strings** per the Version-Bump Checklist (section 2b).
3. **Update CHANGELOG.md** with real content for this version.
4. **Update README.md** first line to reference current version.
5. **Update PROJECT_INSTRUCTIONS.md** change log table.
6. **Run `python tests/preflight.py`** — ALL tests must pass.
7. **Dry-run test**: run `apollo_data_audit.py` to verify it imports without error.
8. **Fill in DEFINITION_OF_DONE.md** header fields (version, date, zip name, builder).
9. **Fill in DELIVERABLES.md** header fields, mark `Changed This Version?` for each file.
10. **Create zip** with ALL files (including DoD + Deliverables).
11. **Extract zip to temp dir** and re-run preflight to verify the zip itself works.

### External Audit Readiness

Builds may be audited at any time by Codex, DeepSeek, Claude, or Gemini. The auditor will:
- Verify every file in `DELIVERABLES.md` exists in the zip.
- Verify no unlisted files exist in the zip.
- Run checks from `DEFINITION_OF_DONE.md`.
- Check version consistency across all files.
- Verify `.bat` files have CRLF line endings.
- Verify `.py` files have no SyntaxError.
- Verify `data_repo/__init__.py` is multi-line (not single-line serialization).

---

## 3. Pre-Flight Test Suite (mandatory since v3.4.2)

No zip gets built until `tests/preflight.py` passes ALL tests on the freshly-extracted zip (not just my working copy).

Current test count: **12 tests**:
1. Syntax check every `.py` file
2. `__future__` import ordering
3. Import every module
4. `app.py` title contains current version
5. `run_apollo.bat` path correctness
6. Real backtest on CYIENTDLM (end-to-end)
7. Backtest determinism (run twice, compare)
8. Bucket classifier does NOT skip
9. All required files present
10. `requirements.txt` valid
11. Universe file paths resolve correctly
12. Version string consistency across files

The test script ships inside the zip — user can run `python tests\preflight.py` to verify on their end.

---

## 4. Bucket Classifier is REFERENCE-ONLY (directive from user)

The bucket classifier is for **display/classification only**, NEVER a gate:
- Every stock reaches the Apollo engine, irrespective of bucket class
- Multiplier has NO effect on stock scores
- Scores come out raw as they should
- The `should_skip` return value was removed in v3.4.1 — never reintroduce it

Guard comment exists in `apollo_core/trade_engine.py` above `total = raw_total`. Do NOT change that line without explicit user approval.

---

## 5. User Skill Level — Novice

The user is not a developer and not tech-savvy. When explaining things:
- Use plain English, no jargon without definition
- Provide step-by-step instructions with exact paths and commands
- Anticipate failure modes (e.g., multiple Python processes, stale cache)
- Include screenshots/diagnostics the user can run themselves
- When in doubt, over-explain rather than under-explain

---

## 6. Phase Roadmap

| Phase | Status | Description |
|---|---|---|
| Phase 1 — Refactor | ✅ Complete (v3.4) | Code restructured into `apollo_core/` + `backtest_engine/` |
| Phase 2 Step 0 — Bucket ungated | ✅ Complete (v3.4.1, v3.4.2) | Gate removed, path bugfixes, pre-flight suite |
| Phase 2 Step 1 — Split `extract_trades()` | ⏳ Pending | `detect_exit_triggers()` (pure) + `execute_exit()` |
| Phase 2 Step 2 — Live engine core | ⏳ Pending | `live_engine/data_replay.py`, `signal_monitor.py`, `state_store.py` |
| Phase 2 Step 3 — Live dashboard | ⏳ Pending | `live_engine/dashboard.py`, `run_live.py` |
| Phase 2 Step 4 — Alert delivery | ⏳ Pending | `live_engine/alert_manager.py` (file log + Telegram) |
| Phase 2 Step 5 — Documentation | ⏳ Pending | `LEARN.md`, `RUNBOOK.md` |
| Phase 3 — VPS + Kite + Telegram | ⏳ Pending | Cloud deployment |
| Phase 4 — Paper trade | ⏳ Pending | Live data, no real orders |
| Phase 5 — Live trade | ⏳ Pending | Real orders |

---

## 7. Test Stocks (locked by user)

The 5 stocks for live simulation testing:
1. APOLLO
2. CYIENTDLM
3. SYRMA
4. JYOTICNC
5. KAYNES

---

## 8. File Paths

| Path | Purpose |
|---|---|
| `/home/z/my-project/download/` | User-facing deliverables (zips, individual files) |
| `/home/z/my-project/scripts/` | Generation scripts (persisted, not delivered) |
| `/home/z/my-project/worklog.md` | Shared multi-agent work log (append-only) |
| `/home/z/my-project/PROJECT_INSTRUCTIONS.md` | THIS FILE — read before every delivery |

---

## 9. Communication Style

- Confirm understanding before each major step
- Provide thorough novice-friendly guidance (like in "Conversation for Next Version.txt")
- Incremental delivery with verification gates
- When bugs are found: fix → bump version → run pre-flight → re-deliver
- Never ship without running the pre-flight suite

---

## 10. DATA GOVERNANCE — Central Repository (mandatory since v4.10)

**All persistent data MUST be stored in APOLLO_DATA_REPOSITORY.**
No database files, caches, reports, or output must be written to the project zip folder.
The project folder is disposable (replaced on each zip extract). Only the data
repository persists across builds.

Central path (Windows):
```
C:\Users\Akhilesh Bhardwaj\Desktop\Trading App Research\GITHUB DATA SOL\APOLLO_DATA_REPOSITORY
```

### Storage Map

| Data | Repository Subfolder | Module | Format |
|------|----------------------|--------|--------|
| EOD price data (CSV/Parquet) | `.` (root) | eod2_loader | CSV / Parquet |
| Backtest history | `history/backtest_history.db` | backtest_history.py | SQLite |
| Guidance: trade records | `output/trades.parquet` | guidance_engine/recorder | Parquet |
| Guidance: per-stock profiles | `output/profiles/SYMBOL.{parquet,md}` | guidance_engine/analyzer | Parquet + Markdown |
| Guidance: MAE/MFE diagnostics | `output/aggregation/mae_mfe_summary.parquet` | guidance_engine/mae_mfe | Parquet |
| Guidance: systemic aggregation | `output/aggregation/aggregation_*.{parquet,md}` | guidance_engine/aggregator | Parquet + Markdown |
| Regime cache (NIFTY/VIX) | `regime/regime_daily.parquet` | guidance_engine/regime | Parquet |
| Live engine state | `live_state/live_state.db` | live_engine/state_store | SQLite |
| Live engine daily reports | `live_state/reports/` | live_engine/daily_report | Markdown + CSV |

### Rules for Future Builds

1. **NEVER** hard-code paths like `Path(__file__).parent / "data"` for persistent storage.
2. **NEVER** write `.db`, `.parquet`, `.csv`, or `.json` output files inside the project folder.
3. All new features that produce persistent data MUST store in a subfolder of
   APOLLO_DATA_REPOSITORY. Choose a descriptive subfolder name.
4. The `DEFAULT_DATA_DIR` in `app.py` (hard-coded to the repository path) is the
   single source of truth. All modules derive their storage paths from it.
5. The central repo path is a constant string, NOT relative to the project folder.
   This ensures it works regardless of where the zip is extracted.

### Analytics Functions (v4.10)

The backtest history DB now supports parameter-based queries for analytics:

- `find_matching_runs(params={}, symbols=[], date_from, date_to)` — find runs
  by parameter combination (e.g., all FIXED_SL runs with SL=5%).
- `get_symbol_history(symbol, params={})` — one stock's results across all
  matching historical runs.
- `get_param_comparison(param_name, param_values, base_params={})` — compare
  aggregate stats grouped by a single parameter's values.
- `migrate_from_legacy(legacy_path)` — one-time copy from old DB location.

### Legacy Migration

On first launch, `app.py` auto-detects a legacy `backtest_engine/data/backtest_history.db`
and migrates all runs to the central repository. The old file is NOT deleted
(you can delete it manually after verifying migration).

### Google Drive Backup (v4.10.1)

A daily sync script (`sync_to_gdrive.py` + `sync_to_gdrive.bat`) is provided
to mirror the APOLLO_DATA_REPOSITORY to Google Drive (15 TB) using rclone.
This enables you to ask AI assistants analytical questions against your data
without needing to write SQL queries yourself.

Setup: see inline docstring in `sync_to_gdrive.py` (one-time rclone config).
Automation: add `sync_to_gdrive.bat` to Windows Task Scheduler (daily run).

---

## Change Log to This File

| Date | Change |
|---|---|
| 2026-07-27 | Initial creation. Captured: file naming convention, title-version sync, pre-flight suite, bucket-ungated rule, novice user, phase roadmap, test stocks, paths, comms style. |
| 2026-07-30 | v4.3: Added version-bump discipline — every engine code change must update version in file headers, app.py title, pre-flight CURRENT_VERSION, CHANGELOG.md, README.md, PROJECT_INSTRUCTIONS.md, and zip filename. RENKO_HARD_GATE=5 added. classify_score() now accepts r_points for Renko Hard Gate. New action "RENKO GATED". |
| 2026-08-01 | v4.7: Added Pool E (Equity Analytics, 21 pts) — E1 52W-Low proximity, E2 Stochastic oversold turn, E3 Relative Volume surge, E4 VPT accumulation, E5 50-SMA recovery zone. Added entry gates G1 PE (etmoney), G2 liquidity floor, G3 52W-low proximity, G4 52W-high distance — enforced at entry (GATE-BLOCKED). New modules gates.py + fundamentals.py. Created tests/preflight.py. |
| 2026-08-06 | v4.10: Integrated Guidance Engine (8-file module: schemas, regime, recorder, mae_mfe, analyzer, flag_generator, aggregator, __init__). Non-invasive post-trade analysis — MAE/MFE diagnostics, per-stock behavioral profiles, regime awareness (NIFTY/VIX), systemic aggregation at 55% threshold. 3 surgical edits to app.py (imports, per-stock recording, post-run pipeline). Output stored in APOLLO_DATA_REPOSITORY/output/. New dependency: yfinance. Data centralization — all persistent storage moved to APOLLO_DATA_REPOSITORY. Backtest history DB, guidance engine output, regime cache, live engine state DB + reports now all stored centrally. Added find_matching_runs(), get_symbol_history(), get_param_comparison() analytics functions. Auto-migration from legacy DB paths. |
| 2026-08-06 | v4.10.1: Bugfix — moved `_generate_xlsx_bytes()` def before tab_results to fix Streamlit module-level NameError. Guidance engine errors now visible in UI (console + status message) instead of silently swallowed. History tab now uses `get_history()` (central repo) instead of `BacktestHistory()` (legacy path). Added Google Drive daily sync script (rclone-based). |
| 2026-08-07 | v4.12: Parquet Central Data Repository — replaced per-symbol CSV reads with `data_repo/` module (sync.py, repo.py, sources.py). Single shared Parquet store at APOLLO_DATA_REPOSITORY. Created `apollo_core/config.py` as single source of truth for APOLLO_DATA_ROOT. `build_universe.py` now uses config path. `eod2_loader.py` imports from data_repo. `run_sync_data.bat` updated with `--repo-dir` and `--update` flags. No more universe filtering — all 3525 CSVs sync. |
| 2026-08-07 | v4.12.1: Bugfix — fixed `data_repo/__init__.py` SyntaxError (triple-quoted docstring serialized as single line by Write tool). Enhanced `repo.py` with case-insensitive date detection, timezone stripping, index name normalisation, memory_map reads, column pruning. Rewrote `sync.py` to remove universe filtering. Fixed `build_universe.py` `--data-dir` default to use config path. |
| 2026-08-07 | v4.12.1+: Added `apollo_data_audit.py` + `run_audit.bat` as **standard build files** — must be included in every future Apollo zip. Audit tool validates CSV format, OHLCV integrity, statistical anomalies, Parquet schema, and CSV-to-Parquet fidelity. Paths hardcoded to user's CSV dir and APOLLO_DATA_REPOSITORY. Generates HTML + JSON reports. |
| 2026-08-07 | v4.12.1+ (process): Added `DEFINITION_OF_DONE.md` + `DELIVERABLES.md` as **mandatory build files**. Every zip must include both, filled in by builder. External AI audit readiness enforced — Codex/DeepSeek/Claude/Gemini may audit any build. Added mandatory 11-step build pipeline to PROJECT_INSTRUCTIONS.md section 2c. |
