# Apollo Engine — Definition of Done (DoD)

**Every feature, fix, or build MUST pass every item below before the zip is shipped.**
No exceptions. No "I'll verify later." If any checklist item fails, the build does NOT ship.
An external AI auditor (Codex / DeepSeek / Claude / Gemini) may verify any build against this checklist at any time.

---

**Build Version**: <!-- SET BY BUILDER -->
**Build Date**: <!-- SET BY BUILDER -->
**Zip Filename**: <!-- SET BY BUILDER -->
**Builder**: <!-- SET BY BUILDER -->
**Auditor**: <!-- SET BY AUDITOR -->
**Audit Date**: <!-- SET BY AUDITOR -->
**Audit Result**: <!-- PASS / FAIL -->

---

## MASTER RULE

> **"Do NOT rewrite proven code. Do NOT modify files you didn't change. Do NOT introduce changes outside the scope of the current task. Test cross-file contracts before delivering."**

The root cause of every regression in past deliveries was violating one or more of these three rules.

---

## Karpathy Guidelines

Behavioral rules to reduce common LLM coding mistakes:

1. **Think Before Coding** — State assumptions explicitly. If uncertain, ask. If multiple interpretations exist, present them. If a simpler approach exists, say so.
2. **Simplicity First** — No features beyond what was asked. No abstractions for single-use code. No speculative "flexibility." If 200 lines could be 50, rewrite it.
3. **Surgical Changes** — Touch only what you must. Don't "improve" adjacent code. Match existing style. Every changed line must trace directly to the user's request.
4. **Goal-Driven Execution** — Transform tasks into verifiable goals. State a brief plan with verify steps. Strong success criteria prevent constant clarification.

---

## SECTION A — Pre-Work (Before Writing Any Code)

### A1. Read the Three Governing Documents
- [ ] `PROJECT_INSTRUCTIONS.md` — Read in full. Note version-bump checklist, pre-flight requirements, phase roadmap, file naming convention.
- [ ] `CHANGELOG.md` — Read the latest version section. Understand what was shipped last.
- [ ] `README.md` — Confirm it matches the current version. Flag if stale.

### A2. Identify Scope
- [ ] Write down EXACTLY which files need to change and WHY.
- [ ] Write down which files must NOT be touched.
- [ ] If the task involves `data_repo/`, `eod2_loader.py`, or `sync.py` — document the full import chain and function signatures before starting.

### A3. Confirm Understanding with User
- [ ] For non-trivial tasks, confirm understanding before starting work.
- [ ] State which files will change and which will be left untouched.

---

## SECTION B — Implementation Rules (While Writing Code)

### B1. Never Rewrite Proven Modules
- [ ] If a module was working in a previous delivery, do NOT rewrite it from scratch.
- [ ] Only make targeted, minimal changes to fix the specific issue.
- [ ] If you must restore a file from a previous zip, also update ALL callers to match.

### B2. Cross-File Contract Integrity
- [ ] Every `import` statement must reference a name that actually exists in the target module.
- [ ] Every function call must match the target function's parameter name, order, and count.
- [ ] Every CLI argument (`.bat` → argparse) must match the `add_argument` dest/flag name.

### B3. Case Sensitivity
- [ ] `_sym_to_eod2()` in `app.py` MUST return `.upper()` (parquet filenames are UPPERCASE).
- [ ] Symbol comparisons must be case-insensitive OR explicitly uppercased.
- [ ] Filenames on disk (`RELIANCE.parquet`) are UPPERCASE — never lowercase.

### B4. No Changes Outside Scope
- [ ] Do not add features, refactors, or "improvements" that were not requested.
- [ ] Do not fix "cosmetic" issues in files unrelated to the task.
- [ ] Do not reformat, reorder, or restyle code you didn't need to touch.

---

## SECTION C — Machine-Auditable Checklist

### 1. Version Consistency

| # | Check | Status | Auditor Note |
|---|-------|--------|-------------|
| 1.1 | `backtest_engine/app.py` HTML title contains current version string | <!-- PASS/FAIL --> | |
| 1.2 | `backtest_engine/app.py` file header comment matches version | <!-- PASS/FAIL --> | |
| 1.3 | `apollo_core/scoring.py` docstring header shows version | <!-- PASS/FAIL --> | |
| 1.4 | `apollo_core/trade_engine.py` docstring header shows version | <!-- PASS/FAIL --> | |
| 1.5 | `apollo_core/constants.py` docstring header shows version | <!-- PASS/FAIL --> | |
| 1.6 | `tests/preflight.py` CURRENT_VERSION matches build version | <!-- PASS/FAIL --> | |
| 1.7 | `CHANGELOG.md` has a section for this version with real content | <!-- PASS/FAIL --> | |
| 1.8 | `README.md` first line references this version | <!-- PASS/FAIL --> | |
| 1.9 | `PROJECT_INSTRUCTIONS.md` change log has entry for this version | <!-- PASS/FAIL --> | |
| 1.10 | Zip filename follows convention: `APOLLO_<TYPE>_<DDMMYY>_v<X.Y.Z>.zip` | <!-- PASS/FAIL --> | |

### 2. Code Quality

| # | Check | Status | Auditor Note |
|---|-------|--------|-------------|
| 2.1 | All `.py` files pass syntax check (no SyntaxError) | <!-- PASS/FAIL --> | |
| 2.2 | All modules import: `apollo_core`, `data_repo`, `backtest_engine`, `guidance_engine` | <!-- PASS/FAIL --> | |
| 2.3 | No `import` statements reference local/dev-only paths | <!-- PASS/FAIL --> | |
| 2.4 | No `print()` debugging statements in production code | <!-- PASS/FAIL --> | |
| 2.5 | All `__future__` imports are at the top of every `.py` file | <!-- PASS/FAIL --> | |
| 2.6 | No triple-quoted docstrings on a single line (causes SyntaxError with `---`/`===`) | <!-- PASS/FAIL --> | |
| 2.7 | All `.bat` files have CRLF line endings (not LF-only) | <!-- PASS/FAIL --> | |

### 3. Import Chain & Function Call Integrity

| # | Check | Status | Auditor Note |
|---|-------|--------|-------------|
| 3.1 | Every `from data_repo` import references a name exported by `data_repo/__init__.py` | <!-- PASS/FAIL --> | |
| 3.2 | Every exported name in `__init__.py` actually exists in the source module | <!-- PASS/FAIL --> | |
| 3.3 | `load_daily` calls use `(repo_dir, symbol)` parameter order | <!-- PASS/FAIL --> | |
| 3.4 | `save_daily` calls use `(repo_dir, symbol, df, source=)` parameter order | <!-- PASS/FAIL --> | |
| 3.5 | Every `.bat` `--flag` has a matching `add_argument` in the Python script | <!-- PASS/FAIL --> | |

### 4. Data Governance

| # | Check | Status | Auditor Note |
|---|-------|--------|-------------|
| 4.1 | `apollo_core/config.py` exists and defines `APOLLO_DATA_ROOT` | <!-- PASS/FAIL --> | |
| 4.2 | No module writes persistent data (`.db`, `.parquet`, `.csv`) to the project folder | <!-- PASS/FAIL --> | |
| 4.3 | All persistent storage paths derive from `APOLLO_DATA_ROOT` | <!-- PASS/FAIL --> | |
| 4.4 | `run_sync_data.bat` `--repo-dir` matches `APOLLO_DATA_ROOT` | <!-- PASS/FAIL --> | |
| 4.5 | `data_repo/__init__.py` has proper multi-line structure (not single-line) | <!-- PASS/FAIL --> | |
| 4.6 | `eod2_loader.py` imports from `data_repo` (not reading CSVs directly) | <!-- PASS/FAIL --> | |

### 5. Testing

| # | Check | Status | Auditor Note |
|---|-------|--------|-------------|
| 5.1 | `python tests/preflight.py` passes ALL tests (0 failures) | <!-- PASS/FAIL --> | |
| 5.2 | A real backtest completes on at least one test stock without error | <!-- PASS/FAIL --> | |
| 5.3 | Backtest produces trade output (not zero trades silently) | <!-- PASS/FAIL --> | |
| 5.4 | `apollo_data_audit.py` runs without error (dry-run validated) | <!-- PASS/FAIL --> | |

### 6. Documentation

| # | Check | Status | Auditor Note |
|---|-------|--------|-------------|
| 6.1 | `CHANGELOG.md` updated with real content for this version | <!-- PASS/FAIL --> | |
| 6.2 | `README.md` reflects current version and feature set | <!-- PASS/FAIL --> | |
| 6.3 | `PROJECT_INSTRUCTIONS.md` change log table has a row for this version | <!-- PASS/FAIL --> | |
| 6.4 | `DEFINITION_OF_DONE.md` exists and its header fields are filled in | <!-- PASS/FAIL --> | |
| 6.5 | `DELIVERABLES.md` exists and lists all files in the zip | <!-- PASS/FAIL --> | |

### 7. Build Completeness

| # | Check | Status | Auditor Note |
|---|-------|--------|-------------|
| 7.1 | All files in `DELIVERABLES.md` exist in the zip | <!-- PASS/FAIL --> | |
| 7.2 | No unlisted files in the zip (except `__pycache__`) | <!-- PASS/FAIL --> | |
| 7.3 | `requirements.txt` lists all runtime dependencies | <!-- PASS/FAIL --> | |
| 7.4 | `apollo_data_audit.py` + `run_audit.bat` are present | <!-- PASS/FAIL --> | |
| 7.5 | `DEFINITION_OF_DONE.md` + `DELIVERABLES.md` are present | <!-- PASS/FAIL --> | |
| 7.6 | All `.bat` files pass `--flag` → argparse verification | <!-- PASS/FAIL --> | |
| 7.7 | `apollo_universe.json` is included and format matches loader expectations | <!-- PASS/FAIL --> | |

### 8. Functional Verification

| # | Check | Status | Auditor Note |
|---|-------|--------|-------------|
| 8.1 | `run_apollo.bat` launches without error | <!-- PASS/FAIL --> | |
| 8.2 | `run_sync_data.bat` points to correct CSV and Parquet repo dirs | <!-- PASS/FAIL --> | |
| 8.3 | `run_audit.bat` has CRLF and calls `apollo_data_audit.py` correctly | <!-- PASS/FAIL --> | |
| 8.4 | No hardcoded `sys.path.insert` or `os.chdir` to dev machine paths | <!-- PASS/FAIL --> | |

### 9. Project-Specific Rules

| # | Check | Status | Auditor Note |
|---|-------|--------|-------------|
| 9.1 | Bucket classifier is REFERENCE-ONLY — never a gate, no `should_skip` | <!-- PASS/FAIL --> | |
| 9.2 | Gates G1-G4 are informational — no `GATE-BLOCKED` in trade_engine/signal_monitor | <!-- PASS/FAIL --> | |
| 9.3 | 5 locked test stocks (APOLLO, CYIENTDLM, SYRMA, JYOTICNC, KAYNES) have parquet data | <!-- PASS/FAIL --> | |
| 9.4 | User is novice — plain English, exact paths, over-explain | <!-- PASS/FAIL --> | |

---

## SECTION D — Zip Packaging

- [ ] Zip filename: `APOLLO_<TYPE>_<DDMMYY>_v<X.Y.Z>.zip`
- [ ] All modified files included, no `__pycache__/` or `.pyc`
- [ ] Delete previous zip versions from the SAME day (keep prior-day zips as history)

---

## SECTION E — Communication

- [ ] State exactly what was changed (files modified, lines changed)
- [ ] State exactly what was NOT changed and why
- [ ] Include any manual steps the user needs to take
- [ ] If a fix required restoring a file from a previous zip, state it explicitly

---

## Quick-Reference: Critical File Contracts

| File | Key Exports / Functions | Notes |
|------|------------------------|-------|
| `data_repo/__init__.py` | `load_daily`, `save_daily`, `list_symbols` | Must be multi-line. ALL names must actually exist in source modules |
| `data_repo/repo.py` | `load_daily(repo_dir, symbol)`, `save_daily(repo_dir, symbol, df, source=)`, `list_symbols(repo_dir)` | repo_dir is FIRST param |
| `data_repo/sync.py` | CLI flags must match `argparse` definitions | Imports from `data_repo.repo` (not `data_repo` directly) |
| `backtest_engine/eod2_loader.py` | `from data_repo import load_daily as _load_parquet_daily` | Do NOT rewrite |
| `backtest_engine/app.py` | `_sym_to_eod2()` returns `.upper()` | Do NOT change case behavior |
| `apollo_core/config.py` | `APOLLO_DATA_ROOT` constant | Single source of truth for data paths |

---

## Audit Instructions (for external AI auditors)

1. Extract the zip to a temporary directory.
2. For each checklist row, run the specified check.
3. Mark Status as `PASS` or `FAIL`.
4. If any row is FAIL, the build does not meet the Definition of Done.
5. Set the Audit Result header to `PASS` (all pass) or `FAIL` (any fail).

Common commands:
```bash
# Syntax check all Python files
find . -name "*.py" -exec python -c "import ast; ast.parse(open('{}').read())" \

# Import check
python -c "import apollo_core; import data_repo; import backtest_engine; import guidance_engine"

# Pre-flight suite
python tests/preflight.py

# CRLF check for .bat files
file run_*.bat

# Line-break check for __init__.py
wc -l data_repo/__init__.py   # should be > 1

# Import chain verification
grep -rn "from data_repo" --include="*.py"
grep -rn "import data_repo" --include="*.py"

# Function call verification
grep -rn "load_daily\|save_daily\|list_symbols" --include="*.py"
```

---

## Build History

| Version | Date | Builder | Auditor | Result |
|---------|------|---------|---------|--------|
| <!-- version --> | <!-- date --> | <!-- builder --> | <!-- auditor --> | <!-- PASS/FAIL --> |
