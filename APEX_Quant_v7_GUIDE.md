# APEX Quant v7 — Guide

File: `APEX_Quant_v7.pine` (747 lines; last line `// ═══ END OF APEX QUANT v7 ═══`).

## 1. How a higher-timeframe signal reaches your 1m chart

```
 15m candle 09:45–10:00 closes
        │
        ▼
 f_engine() runs on 15m bars (same code the 15m chart runs)
        │  → BUY, Entry / SL / TP decided on the 15m close
        ▼
 request.security("15", f_engine()[1], lookahead_on)   ← confirmed bar only, no repaint
        │
        ▼
 1m chart, first tick of the 10:00 candle (timeframe.change("15") = true)
        │  → orange tag "15m BUY" + dashed red SL / blue TP lines
        │  → alert fires immediately (once, via its own alert() call)
        ▼
 Sniper ON:  wait ≤ 30 × 1m bars for a pullback to 50% of the 15m signal candle,
             then a confirming 1m close → "◆" fill (better entry, same SL / TP)
             no pullback / SL or TP touched first → tag gets "✗" (missed, no chase)
        ▼
 Every confirmed 1m candle: SL / TP checked → lines stop on the exact 1m candle that touches
```

The tag appears at the HTF candle's close (10:00 for the 09:45–10:00 candle). Showing it any earlier would require repainting.
On the 15m chart itself, the same BUY sits on the 09:45 candle with the same Entry / SL / TP. The engine is identical and runs on the same 15m bars.
Caveat: a 1m chart loads less history. The first mirrored signals, until about 1,300 15m bars have loaded, can differ slightly while the EMA-200 warms up.

## 2. Inputs
| Group | Input | Default | Range | Purpose |
|---|---|---|---|---|
| Engine | Mode | Auto | Auto / Intraday / Swing | Per timeframe: ≤ 1h intraday, 4H+ swing |
| | Signal frequency | Normal | Active / Normal / Conservative | Regime thresholds, cooldowns, duplicate window |
| | Direction | Auto | Auto / Both / Long only | Swing = long only in Auto |
| | Timezone | GMT+4 | text | Times, day reset, session |
| | Trend / Range / Swing-BO setups | on | bool | Enable each setup |
| | Anti-chase | 1.2 ATR | ≥ 0.3 | Max distance from EMA-21 |
| | Near 52-bar high | 25 % | 1–100 | Swing leadership filter |
| | Swing BO volume | 1.5 × | ≥ 0.5 | Breakout volume |
| Risk | TP intraday / swing | 1.3 R / 2.5 R | 0.5–10 | Trend-setup target |
| | Range min / cap R | 0.8 / 2.0 | — | Mid-band target limits |
| | SL floor × ATR | 1.2 | ≥ 0.3 | Stop outside bar noise |
| | SL floor × avg 8-bar range | 0.5 | ≥ 0 | Stop outside multi-bar swings (1m fix) |
| | SL buffer | 0.3 ATR | ≥ 0 | Beyond the swing |
| | Skip if SL wider than | 3 ATR | ≥ 1 | Bad location → skip |
| | Spread ticks intraday / swing | 30 / 2 | ≥ 0 | Deducted from every trade |
| | Partial 50% at 1R + BE | off | bool | Lower variance; not higher expectancy |
| | Time-stop | auto | bars | 1m 90, 5m 48, 15m 32, 1h 24, swing 60 |
| Session | Session filter | Auto | Auto / On / Off | On for 1m–5m signal TFs |
| | Session | 11:00–01:00 | session | London open → NY close (GMT+4) |
| | Max trades / day / TF | 10 | 1–50 | Intraday cap |
| MTF | Mirror TF 1 / 2 / 3 | 15 on / 60 on / 240 off | timeframe | Only TFs above the chart TF are used |
| | Aligned-only filter | Auto | Auto / On / Off | Auto = on for 1m / 5m charts |
| | Sniper entry | on | bool | Pullback fill for mirrored signals |
| | Sniper max wait | 30 bars | 2–500 | Then "missed" |
| Alerts | Chart / TF1 / TF2 / TF3 / exits | on | bool | Mute any source |
| Display | Keep last N per source | 15 | 1–50 | Chart cleanliness |
| | MAE/MFE analysis source | Chart TF | 4 sources | Which trades the calibration table uses |

## 3. Setup
1. Pine Editor → new indicator → paste the whole file → Save → Add to chart. Remove older APEX versions.
2. Indicator name → ⋯ → **Pin to scale → Pin to right scale**.
3. Alert → Condition **APEX Q7 → Any alert() function call** → tick **Show pop-up** + **Play sound** → Create.
4. Reading the chart:
   - Green/red **BUY / SELL** = chart-TF signal; **A+** = aligned with the mirrored timeframes.
   - Orange **15m BUY**, purple **1H SELL**, teal **4H …** = mirrored signals with their own dashed SL / TP.
   - **◆** = sniper fill; **✗** = missed (no pullback).
5. Trade table (top right): chart trade, each mirrored TF's bias (↑ / ↓) and open trade, and **HTF alignment** (ALIGNED ↑ / ↓ or MIXED).
6. Stats (bottom right): one row per source. Win % is shown with its break-even %, plus PF, expectancy, SL-first %, and missed sniper orders.
7. MAE / MFE (bottom left):
   - % of trades that reached 0.5 / 1 / 1.5 / 2R before the SL.
   - **Suggested TP**: the target that would have produced the most R per trade after costs.
   - **Stop check**: "SL too tight" when winners typically came ≥ 0.7R close to the stop.
   - Ignore the table under 100 trades.

## 4. Why the SL gets hit before the TP, and what v7 does about it
- **Entering late, after the move** → the pullback-only setups and anti-chase filter remain; the sniper fill buys the dip inside the HTF signal candle instead of its close. The same SL then means a smaller loss and a larger R.
- **Stop inside normal noise** → floor = max(1.2 ATR, 0.5 × average 8-bar range) plus a buffer beyond the swing. The Stop check row tells you from your own data if it is still too tight.
- **Target further than price usually travels** → the MFE rows show how far trades really go. Suggested TP picks the target that maximises net R (conservative: it never assumes profits beyond an exit that actually happened).
- **Trading against the bigger timeframe** → the A+ / aligned-only filter (on by default for 1m / 5m).
- **Trade-offs:**
  - A smaller TP raises win % but lowers R per win, so only expectancy / PF tell you whether it helped.
  - The sniper improves entries but misses some trades (see the Missed column).
  - The aligned-only filter removes trades too.
  - Partials smooth the equity curve but never raise expectancy.

## 5. Notes / known limits
- Mirrored trades start being tracked from the candle after the tag. The signal price is the HTF close, which is effectively this candle's open.
- If a mirrored trade is still open when that timeframe fires again, it is closed at market (counted as TIME).
- Research tool. Judge it only on 100+ trades per source, out-of-sample, after costs.
