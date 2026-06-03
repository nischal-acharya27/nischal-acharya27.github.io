# Changelog

All notable changes to SpikeWatch.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
The project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

The bump rule for this repo:
- **MAJOR** — breaking on-disk state changes (DB schema not auto-migrated,
  removed strategies, removed UI features).
- **MINOR** — new features (new strategy, new trade-store column, new UI panel).
  Safe upgrades.
- **PATCH** — bug fixes and internal refactors with no behaviour change.

`src/__init__.py` holds the single source of truth (`__version__`).

---

## [0.3.0] — 2026-05-23

### Added
- **Backtest JSON export** (`src/data/backtest_export.py`) — Export Backtest…
  button in the backtest panel writes a self-contained JSON file containing
  the run config, every closed trade (ISO timestamps), the equity curve
  (downsampled to ~600 points with drawdown alongside), a per-month
  aggregation, and the close-reason histogram. Schema is versioned and
  pinned by 11 tests so downstream consumers can rely on a stable shape.
  Atomic write (.tmp + rename) so a crash mid-write never leaves a
  truncated file behind.

### Notes
- Marketing site under `site/` reads the same export format directly.
  Run a backtest, click Export, drop the file in `site/data/` and the
  showcase numbers refresh.

---

## [0.2.0] — 2026-05-23

This is the first version after the architectural overhaul recorded in
`architecture.json`.  87 tests now back the codebase (was 0).

### Added
- **Pluggable strategy framework** (`src/core/strategy_base.py` + `strategies_pkg/`).
  Adding a strategy is one new file in `strategies_pkg/`; no engine or UI edits.
- **SQLite-backed trade store** (`src/data/trade_store.py`).
  Lifetime trade history persists across launches with full-filter API
  (symbol substring, exchange, close reason, date range).  CSV mirror kept
  for backward compatibility and one-shot import from legacy CSVs.
- **Header-driven rate limiter** (`src/data/rate_limit.py`).
  Throttles before hitting Binance's `X-MBX-USED-WEIGHT-1M` or Bitget's
  `X-RateLimit-Remaining` cap.  Exponential 429 backoff that auto-resets on 2xx.
- **Sortino and Calmar ratios**, alongside a rewritten Sharpe based on log
  returns of a resampled equity curve.  Crypto 24/7/365 annualisation
  (previously the legacy code used a 252-trading-day equity-market calendar).
- **QSettings persistence**: threshold, TP/SL, trailing SL, EMA short/long,
  MA period, std-dev multiplier, timeframe, last strategy, window geometry.
- **Status bar** with per-exchange last-sweep telemetry (cycle, symbol
  coverage, elapsed time), colour-coded by success ratio.
- **Lifetime closed-trades view**: the engine preloads the last 2000 closed
  trades from SQLite on startup, so the closed-trades panel no longer
  resets every launch.
- `requirements.txt` pinning PyQt5, pyqtgraph, numpy, requests, aiohttp, pytest.
- `architecture.html` + `architecture.json` (interactive map + agent-readable
  spec) at the repo root.
- 87 passing unit tests across 9 test files (`tests/`).

### Changed
- **Live monitoring is now ~10× faster.** Single background asyncio loop
  with one `aiohttp.ClientSession` per exchange and `Semaphore(20)`
  concurrency.  A ~200-symbol Binance sweep dropped from ~50 s to ~3–5 s.
- **One unified TP / SL / trailing-SL state machine** (`src/core/trade_state.py`)
  used by both the live engine and all three backtest simulators.  Live and
  backtest can no longer drift.
- The three `simulate_*_trades` functions are now thin shims that delegate
  to `strategy_base.run_backtest`.  522 LOC of duplication collapsed to 220.
- `src/core/utils.py:calculate_advanced_metrics` returns a `MetricsResult`
  dataclass instead of a 3-tuple.  Backtest summary log now shows
  annualised return, Sharpe, Sortino, Calmar, drawdown, profit factor.
- Engine open-trade state typed as `dict[str, Trade]` (was `dict[str, dict]`).
  UI reads `trade.direction` etc. via attribute access.
- All errors now flow through Python's `logging` module and Qt log signals.
  No more `print()` in the data layer; no more bare `except:` in the UI.

### Removed
- **`SpikeWatch.py` monolith** moved to `legacy/`.  README's PyInstaller
  build command now targets `main.py`.
- Duplicated Bitget timeframe map (was in `engine.py` AND `exchange_api.py`).
  Single source of truth in `data/exchange_api.py`.
- Inline pagination logic from `engine.execute_backtest`.  Now lives in
  `data/exchange_api.paginate_historical_klines`.
- Engine's `_init_trade_history_file` / inline CSV writes.  TradeStore owns
  the header creation and writes.

### Fixed
- `np.arange` dropped the equity curve's final point (broke max drawdown
  on certain backtest lengths).
- `math.exp` overflowed on high-velocity backtests (now caught and surfaced
  as `+inf` or `-100`).
- 429 backoff timer wasn't cleared by the next 2xx response, gating calls
  for tens of seconds for nothing.
- Bare `except:` in `_fetch_all_symbols_at_startup` swallowed network errors
  silently.  Now raises a `QMessageBox` and logs.

### Architecture
- One background daemon thread runs one asyncio loop.  Was: three daemon
  threads each running their own `asyncio.run()` with blocking `requests`.
- Closed-trade records remain plain dicts — they're the persistence
  boundary (SQLite rows, CSV mirror, Qt signal payloads).

---

## [0.1.0] — pre-2026-05-23 (legacy)

Original single-file `SpikeWatch.py` plus an in-progress `src/` modular
rewrite.  Both shipped in the repo simultaneously; the README packaged
the monolith while `main.py` ran the modular version.  This version
exists for historical reference — preserved in `legacy/SpikeWatch.py`.
