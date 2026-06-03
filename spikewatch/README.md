# SpikeWatch

Current version: **0.2.0** — see [CHANGELOG.md](CHANGELOG.md) for what changed.

A PyQt5 desktop app that monitors price spikes on Binance and Bitget USDT
perpetual-futures markets, opens simulated (paper) trades on spikes with
configurable take-profit / stop-loss / trailing-stop, and backtests three
strategies (Spike Detection, EMA Crossover, Simple Mean Reversion) against
historical klines.

> **Paper trading only.** SpikeWatch reads public market data; it does not
> place real orders on any exchange.

## Run

```
python main.py
```

Requires Python 3.10+ and the packages in `requirements.txt`
(PyQt5, pyqtgraph, numpy, requests).

```
pip install -r requirements.txt
```

## Build a standalone binary

```
pyinstaller --windowed --onefile main.py
```

The PyInstaller output lives in `dist/`.

## Project layout

```
main.py                 # Entry point: QApplication + SpikeWatchApp
src/
  ui/
    main_window.py      # Top-level window, owns the engine
    widgets.py          # TradeItemWidget (open-trade row)
    charts.py           # PriceChartWindow, EquityCurveChartWindow
  core/
    engine.py           # TradingEngine: monitoring + backtest orchestration
    signals.py          # SignalEmitter (Qt signal bus)
    strategies.py       # Pure-function backtest simulators
    utils.py            # Equity curve, Sharpe, drawdown, profit factor
  data/
    exchange_api.py     # Binance + Bitget public REST clients
legacy/
  SpikeWatch.py         # Pre-refactor single-file implementation; not used.
architecture.html       # Interactive architecture map (open in a browser)
architecture.json       # Machine-readable view of the same map
demo_trade_results.csv  # Append-only trade log written by the engine
```

## Architecture

See `architecture.html` (interactive) or `architecture.json` (structured)
for a full module map, data flows, and a list of known architectural
issues being worked through.
