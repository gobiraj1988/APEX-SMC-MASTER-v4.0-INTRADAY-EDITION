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
The script pins itself to the price scale (`scale = scale.right`) and plots invisible high/low
"scale anchors", so even on its own scale it fits the same range as the candles.
TradingView keeps the scale of an instance that is already on the chart, so after updating:
1. Remove **every** APEX SMC instance from the chart (including the old v4.0).
2. Add it again from Indicators → My scripts.
3. If labels are still off: indicator name → ⋯ → **Pin to scale → Pin to right scale**.

---

# APEX Quant v5 (new, not SMC) — `APEX_Quant_v5.pine`
Trend-pullback + breakout system for intraday (1m–1h, e.g. XAUUSD) and swing / investment (4h, D, W, e.g. CSE Sri Lanka stocks).
- Regime: EMA 21/55 stack + higher-TF EMA-50 + ADX (+ VWAP side intraday, EMA-200 in swing).
- Entry A: pullback to EMA-21 then a momentum candle closing above/below the previous candle.
- Entry B: 20-bar breakout with volume expansion. No-chase filter.
- One SL (red) + one TP (blue); lines stop at the touching candle. Time-stop. Up to 10 trades/day intraday.
- Swing / Invest mode is long-only by default.
- Stats: win %, the break-even win % for the chosen TP, profit factor, net R.
After adding: indicator ⋯ → Pin to scale → Pin to right scale.
