# APEX Quant v6 — Guide

File: `APEX_Quant_v6.pine` (509 lines; last line `// ═══ END OF APEX QUANT v6 ═══`).

## 1. Why v5 hit SL so often → what v6 changes
| v5 problem | v6 change |
|---|---|
| Most trades end at SL | Trades only in a confirmed regime (ADX + ADX slope + Choppiness + ATR percentile); stands aside in chop / volatility spikes. SL floor raised to 1.2 × ATR / 0.5 × HTF-ATR, plus a 0.3 ATR buffer beyond the swing. |
| BUYs near local highs (late) | Entry only after a pullback (EMA-21 touch or 38–62% retracement) + confirmation close; anti-chase: skip if > 1.2 ATR from EMA-21. The v5 intraday breakout entry is removed. |
| Back-to-back duplicate BUYs | No same-direction signal within 8 bars, or within 1 ATR of the last entry (32 bars); cooldown 3 bars after an exit, 6 after a stop-out. |
| 1m SL/TP too tight | HTF-ATR floor (0.5 × 15m ATR on 1m); session filter ON by default for 1m/5m (London open → NY close). |
| ~1 trade/day on 15m | New Range-fade setup (Bollinger 20/2 extreme + RSI(2) exhaustion + rejection wick → mid-band target) trades the sideways hours that trend setups skip; daily cap 10. |

## 2. Inputs
| Input | Default | Range | Purpose |
|---|---|---|---|
| Mode | Auto | Auto / Intraday / Swing | Auto: ≤ 1h intraday, 4H/D/W swing |
| Signal frequency | Normal | Active / Normal / Conservative | Regime thresholds, cooldowns, duplicate window |
| Direction | Auto | Auto / Both / Long only | Auto: swing = long only |
| Your timezone | GMT+4 | text | Signal time, day reset, session |
| Trend pullback setup | on | bool | Continuation entries in trends |
| Range fade setup | on | bool | Mean-reversion entries in ranges (intraday) |
| Swing breakout setup | on | bool | 20-bar breakout + volume (swing) |
| Anti-chase | 1.2 ATR | ≥ 0.3 | Max distance of entry from EMA-21 |
| Swing: near 52-bar high | 25 % | 1–100 | Leadership filter |
| Swing breakout volume | 1.5 × | ≥ 0.5 | Volume confirmation |
| TP intraday trend | 1.3 R | 0.5–10 | Single take-profit |
| TP swing | 2.5 R | 0.5–10 | Single take-profit |
| Range fade min R / cap R | 0.8 / 2.0 | — | Skip small-reward fades; cap mid-band target |
| SL floor × ATR | 1.2 | ≥ 0.3 | Stop outside noise |
| SL floor × HTF ATR | 0.5 | ≥ 0 | Low-TF stop floor |
| SL buffer | 0.3 ATR | ≥ 0 | Beyond the swing (or spread if larger) |
| Skip if SL wider than | 3 ATR | ≥ 1 | Bad location → skip |
| Spread / cost ticks | 30 intraday / 2 swing | ≥ 0 | Deducted from every trade in the stats |
| Break-even at R | 0 (off) | ≥ 0 | Optional |
| Bank 50% + trail | off | bool | Optional runner; trail = 2 ATR |
| Time-stop | auto | bars | 1m 90, 5m 48, 15m 32, 1h 24, swing 60 |
| Session filter | Auto | Auto / On / Off | Auto = on for 1m–5m, session 11:00–01:00 GMT+4 |
| Max trades/day | 10 | 1–50 | Intraday cap |
| Alerts | on / on | bool | Entry / exit alert() messages |
| Keep last N trades | 20 | 1–100 | Chart cleanliness |

## 3. Setup
1. Pine Editor → new indicator → paste the whole file → Save → Add to chart.
2. Indicator name → ⋯ → **Pin to scale → Pin to right scale**. Remove old APEX versions from the chart.
3. Alert → Condition **APEX Q6 → Any alert() function call** → tick **Show pop-up** + **Play sound** → Create. To silence: pause the alert or untick the alert inputs.

## 4. Research test plan
- **Split:** in-sample = oldest 60 % of loaded history (tune here only); out-of-sample = newest 40 % (use Replay or a date filter; never re-tune on it).
- **Walk-forward:** XAUUSD 1m, 5m, 15m, 1h and 3 liquid CSE stocks (e.g. JKH, COMB, HNB) on D and W. Same settings everywhere, then Active / Normal / Conservative.
- **Record per test:** trades, trades/day, win % vs break-even %, profit factor, expectancy (R, net of cost), max DD (R), max losing run, exit mix (SL / TP / BE / time).
- **"No edge" means:** out-of-sample expectancy ≤ 0 or PF < 1.1 after costs, or results that only hold on one timeframe / one period, or fewer than 100 trades (inconclusive, not proof).
