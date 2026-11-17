# Fork notes (KamranPA/passivbot)

Branch `dev/safety-and-tooling` — changes vs. the previous fork state:

- Merged `enarjord/passivbot` master (fork was 13 commits behind).
- Both workflows now install **this repo's code** (not a fresh upstream clone).
- `passivbot-live.yml`: fixed `relaunch` (it called a non-existent `passivbot.yml`), relaunches only after
  a normal full session, stops on crash and sends a Telegram alert (secrets `TELEGRAM_BOT_TOKEN`,
  `TELEGRAM_CHAT_ID`, optional), runs an OKX coin availability pre-flight check, config selectable via input.
- `passivbot-backtest.yml`: config selectable via input, new `summary` job aggregates all windows
  (compounded gain, worst window, worst drawdown) into the run's Summary tab.
- `configs/my_config_safe.json`: same strategy parameters as `my_config.json`, only risk changed:
  long HSL on, both exposure enforcers on, long TWE 1.5 -> 1.0, leverage 10 -> 5.
- `scripts/check_okx_coins.py`, `scripts/aggregate_backtests.py`: helper scripts.

Suggested order: run the backtest workflow with `my_config.json` (label `base`) and `my_config_safe.json`
(label `safe`) over the same windows, compare the summaries, and only then start live with a small balance.

## Backtest variants (added after first results)
First results (4 windows, 2026-01-01..2026-11-15, OKX): base compounded +35.4% (worst DD 22.5%), safe +10.8%
(worst DD 14.7%). Safe lost 9.4% in 2026-04..07 where HSL (red_threshold 0.15) fired 10 times.
To separate the effect of HSL from the exposure cap:
- `configs/my_config_safe_nohsl.json`: safe config with long HSL off (enforcers + TWE 1.0 + leverage 5 stay).
- `configs/my_config_safe_hsl25.json`: safe config with long HSL red_threshold 0.15 -> 0.25.
Run the backtest workflow with the same windows and labels `safe_nohsl` / `safe_hsl25`.
Note: `live.leverage` (exchange setting) does not change backtest results; TWE does.

## Second round (2025 quarterly + 2026 quarterly, 8 windows per variant)
2025: base +49.6% (worst window -14.2%, DD 57%), safe_nohsl +2.6% (worst -30.9%), safe_hsl25 +14.1% (worst -23.2%).
2026: base +34.2%, safe_nohsl +24.4%, safe_hsl25 +12.8%. No liquidations. HSL helped in 2025-Q1, hurt in 2026-Q2.
Hypothesis (unverified): exposure enforcers (reduce_overweight) realize losses in a crash and explain why safe_nohsl
did worse than base in 2025-Q1. Test: `configs/my_config_safe_noenf.json` = base with ONLY long TWE 1.5 -> 1.0
(HSL off, both enforcers off, leverage 5). Compare against base, safe_nohsl.

## Trade-by-trade findings (base/nohsl, 8 quarterly windows)
- ~99% of closed position cycles are profitable; avg profit per cycle is tiny ($0.04-0.6 on 1000).
- Losses in bad windows (2025-Q1, 2026-Q4) are almost entirely UNREALIZED bags at window end
  (2025-Q1 base: realized +64, equity change -142). Each window starts flat and is marked to market at the end.
- Deep cycles (>=7 entries) are ~4% of cycles and net ~0; `close_unstuck` realized -1023 (base) over 8 windows.
- WLD/NEAR/ZEC/AAVE/ARB are the biggest losers; HYPE/ONDO/INJ/FIL the biggest winners (hindsight, not a rule).
Test 1: continuous 6-month backtests (window_months=6) to see bag recovery.
Test 2: `configs/my_config_depthcap.json` = base with long `we_excess_allowance_pct` 0.37 -> 0.0
(per-coin exposure cap 1.5/7 = 0.214 instead of 0.294), everything else identical to my_config.json.
