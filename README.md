# APEX SMC Master v4.1 — Intraday Edition (TradingView Pine v6)

File: `APEX_SMC_Master_v4.1.pine` — paste into the Pine Editor → **Add to chart**.

## Alerts with sound + pop-up (PC)
1. Alert (⏰) → **Condition:** `APEX SMC` → **Any alert() function call**.
2. **Notifications:** tick **Show pop-up** and **Play sound** (choose a sound, e.g. repeat 1–3×). Optionally tick *Send push/email*.
3. Expiration: *Open-ended*. Create.
4. To stop: pause the alert in the Alerts panel, **or** untick *Send BUY / SELL alerts* in the indicator settings.

The alert message contains symbol, TF, Entry, SL and TP. Exit alerts (TP / SL / time-stop) can be turned off with *Also alert when TP / SL / time-stop hit*.

## What's new in v4.1
- Non-repainting HTF (confirmed HTF bar only) — live signals match the backtest.
- CHoCH must close beyond structure with a displacement candle (body ≥ 0.6 ATR).
- HTF EMA-50 trend must agree with HTF structure bias.
- SL at the sweep extreme + buffer, at least max(1 × chart ATR, 0.5 × HTF ATR) away; skipped if wider than max(3 × ATR, 2 × min SL).
- One TP (default 1.5R). Only 2 lines per trade (SL red, TP blue) — they stop at the candle that touches either.
- Time-stop per TF so no trade runs for days; session filter 10:00–22:00 GMT+4 on by default.
- Chart keeps only the last N trades; CPR / OB box drawings off by default.
- HTF pairing: 1m/3m→15m, 5m→60m, 15m→240m, 1h→240m.

## Drawings not sitting on the candles?
The script pins itself to the price scale (`scale = scale.right`). If an older copy was added before this fix,
remove it from the chart and add it again, or: indicator name → ⋯ → **Pin to scale → Pin to right scale**.
Also remove the old v4.0 indicator from the chart.
