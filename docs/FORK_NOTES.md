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
