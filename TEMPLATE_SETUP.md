# BO Visual Enhancer — Installation & Template Setup

## 1. Install the indicator

1. Open **MetaTrader 4**.
2. Menu: **File → Open Data Folder**.
3. Navigate to `MQL4\Indicators\`.
4. Copy `BO_VisualEnhancer.mq4` into that folder.
5. Return to MT4. In the **Navigator** panel (Ctrl+N) right-click **Indicators → Refresh**.
6. The indicator **BO Visual Enhancer** now appears under Custom Indicators.

## 2. Compile (if needed)

1. Right-click the indicator in Navigator → **Modify**. MetaEditor opens.
2. Press **F7** to compile. `0 errors, 0 warnings` should be shown.
3. Close MetaEditor.

## 3. Attach to chart

1. Open a clean chart of your pair (e.g. EURUSD M1).
2. Drag **BO Visual Enhancer** from Navigator onto the chart.
3. In the input dialog, verify **Allow DLL imports / external expert** is NOT required (this indicator has no imports).
4. Click **OK**. The dark theme, custom candles, dashboard, and signal highlights activate immediately.

## 4. Add your existing RSI + Bollinger Signals indicator

1. Drag your existing signal indicator on top of the chart.
2. This visual enhancer does **not** draw signal arrows — it only paints the chart background, candles, dashboard, and highlight boxes. Your arrows remain fully visible on top.

## 5. Save as template

1. Right-click anywhere on the chart → **Template → Save Template…**.
2. Save as `BO_Dark.tpl` in the default folder that opens (`MQL4/..profiles../templates/` — just accept the default).
3. To set it as the DEFAULT template for every new chart, save it with the exact name **`default.tpl`** in the same folder.

## 6. Apply to any chart

- Right-click chart → **Template → Load Template → BO_Dark**.
- Or, to have every new chart auto-apply it, use filename `default.tpl`.

## 7. Input tuning quick-reference

| Input group          | When to touch                                                                 |
|----------------------|-------------------------------------------------------------------------------|
| `CANDLE DESIGN`      | Turn on Heiken Ashi for smoother trend reading on M1/M5.                      |
| `THEME` / `COLORS`   | All colors are live-editable without recompile.                               |
| `SIGNAL LOGIC`       | Match `InpRSIPeriod`, `InpRSIOverbought/Oversold`, and `InpBBPeriod/Deviation` to your external indicator so the dashboard **SIGNAL** row mirrors its arrows. |
| `TREND FILTER`       | Raise `InpTrendATRMult` if market reads TREND too often on ranging pairs.     |
| `DASHBOARD`          | Move panel via `InpDashX` / `InpDashY`. Disable entirely with `InpShowDashboard = false`. |
| `PERFORMANCE`        | Lower `InpMaxVisualBars` from 300 → 150 if you run many charts on low-end HW. |

## 8. Removal / reset

- Remove indicator: right-click chart → **Indicators List** → select **BO Visual Enhancer** → **Delete**. Theme restores automatically on deinit.
- Clean remnant objects (if any): **Charts → Objects → Object List (Ctrl+B)** → filter `BOVE_` → Delete all.
