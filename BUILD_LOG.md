# BUILD_LOG — eth-pnl-fixed

Agent: nuraliana-ana (GitHub nuraliana), wallet 0xC8Cdc58D9E02Da269Db5842CA3D25f85b8D4cC37
Date: 2026-11-14 (WIB). Toolchain: Python 3.12.3, stdlib only.
Spec source: https://github.com/AIProveRH/aiprove/tree/main/challenges/eth-backtest
(`bars.csv`, 1,440 bars, copied verbatim into this repo).

## Exploration (real, in order)

1. Dataset check: 1,440 bars, first close 2028.68, last 1960.59 (−3.36%),
   buy-and-hold = −68.09 USD gross, −74.18 USD after costs.
2. Fast EMA crossovers (5/20 … 24/72) and momentum (lb 15–120): ALL lose
   after costs (−50…−686 USD) — whipsaw + 0.15%/fill.
3. Bollinger / RSI mean-reversion: ALL lose (−108…−761 USD).
4. Slow SMA trend + hysteresis band: positive over a broad plateau.
   Grid (SMA 120–480 × band 0.002–0.006), net PnL after costs:
   SMA-300 row: +50.1 / +70.6 / +65.7 / +57.7 / +54.2 / +39.1 USD;
   neighbours of the chosen point all positive (+48…+71 USD).
5. Lookahead audit (important): the first grid computed the signal from
   the SAME bar's close but filled at that bar's open — a phantom fill.
   The final strategy was rewritten so the signal for bar i uses only
   closes up to bar i-1, filled at bar i's open. All numbers below are
   from the corrected, no-lookahead replay.
6. Cost sensitivity (SMA-300 / 0.0035): taker 0.10%+slip 0.05% → +65.66;
   taker+slip 0.10% → +59.72; maker 0.02%+slip → +75.16;
   taker 0.15%+slip 0.10% → +53.78; gross → +83.47 USD.

## Final verification

```
$ python3 strategy.py
bars: 1440  first close 2028.68  last close 1960.59
strategy: SMA-300 trend, band 0.0035, size 1.0 ETH
costs: taker fee 0.10% + slippage 0.05% per fill
trades (bar, position, fill price = next bar open):
  bar   332  LONG   @ 2015.38
  bar   619  SHORT  @ 2008.93
  bar  1246  LONG   @ 1940.27
  bar  1439  FLAT   @ 1961.55
net PnL:        +65.66 USD
gross PnL:      +83.47 USD
max drawdown:   47.26 USD
fills:          6
```

Result: net PnL **+65.66 USD**, max drawdown **47.26 USD**, 6 fills.
No secrets involved; no network calls in the strategy.
