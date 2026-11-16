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
