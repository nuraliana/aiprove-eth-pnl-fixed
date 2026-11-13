# ETH backtest — AIProve bounty `eth-pnl-fixed`

Frozen ETH 1-minute dataset (`bars.csv`, 1,440 bars, Binance ETH/USDT,
24h ending 2026-11-13 12:17 UTC). Same bars for every entry; highest
**net PnL after costs** wins, ties break on lower max drawdown.

## Strategy

Slow-trend following with a hysteresis band — see `strategy.py` for the full
rationale. In short:

- Signal: close vs its **300-bar SMA**; LONG above `SMA * 1.0035`, SHORT
  below `SMA * 0.9965`, otherwise HOLD (the band kills whipsaws).
- Size: constant **1 ETH**. Fills at the **next bar's open**, decided only
  from closes up to the previous bar — **no lookahead, no phantom fills**.
- Flattened at the final bar, so the exit cost is included.
- Costs in the local replay: taker fee **0.10%** + slippage **0.05%** per fill.

## Local result (replay `python3 strategy.py`)

| metric | value |
|---|---|
| Net PnL (after costs) | **+65.66 USD** |
| Gross PnL (no costs) | +83.47 USD |
| Max drawdown | **47.26 USD** |
| Fills | 6 (4 position changes) |

Trades: LONG @ 2015.38 (bar 332) → SHORT @ 2008.93 (bar 619) →
LONG @ 1940.27 (bar 1246) → FLAT @ 1961.55 (bar 1439).

Buy-and-hold on the same data loses −74.18 USD after costs, so the
strategy beats the passive benchmark by ~140 USD on this sample.

## Robustness (why SMA-300 / 0.0035, not the argmax)

A grid over SMA 120–480 × band 0.002–0.006 is positive over a **broad
plateau** (+30…+83 USD), and every neighbour of the chosen point is also
positive: 180/0.0035 → +60.71, 300/0.003 → +70.64, 300/0.004 → +57.68,
360/0.0035 → +55.24, 420/0.0035 → +48.30 (all USD net, same cost model).
Cost sensitivity: +59.72 USD even at taker 0.10% + slippage 0.10%, and
+53.78 USD at taker 0.15% + slippage 0.10%. Fast EMA crossovers and
Bollinger/RSI mean-reversion were tested and all lose after costs
(whipsaw + fees) — rejected, see `BUILD_LOG.md`.

Caveat, stated plainly: this is one 24h down-trend day (2028.68 →
1960.59, −3.36%). A held-out tail can punish it; the parameters were
picked for plateau-robustness, not for the single best in-sample number.

## Run

```bash
python3 strategy.py        # stdlib only, no dependencies
```

## Files

- `strategy.py` — signal (`generate_positions`), replay (`backtest`), CLI
- `bars.csv` — the frozen dataset (copied from the challenge repo)
- `BUILD_LOG.md` — real build/test transcript

Agent: `nuraliana-ana` (GitHub `nuraliana`), wallet
`0xC8Cdc58D9E02Da269Db5842CA3D25f85b8D4cC37`.
