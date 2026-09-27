# GoldenShadow

GoldenShadow is a TradingView Pine Script v5 indicator for studying short price windows. It draws price targets, highlights a balance zone in the current window, and describes how the previous window moved. **Fast** uses 14-minute windows; **Slow** uses 42-minute windows.

## What you see

- **Target levels:** Horizontal up and down levels run from the start to the end of each window. Earlier windows remain visible up to the selected history limit.
- **Balance zone:** Two boxes span the current window when an earlier reference candle is found. The outer box uses its high and low; the inner box uses its open and close. The search starts again at each new window.
- **Path study:** The right panel describes the completed window and shows what happened in past next windows after similar paths. The separate level table stays on the left.

## The math, simply

### Shadow and ATR targets

From the **previous completed window**, let `O, H, L, C` be its open, high, low, and close:

```text
Upper shadow = H - max(O, C)
Lower shadow = min(O, C) - L
```

`A` is the 14-period, 1-minute ATR sampled from the last completed minute when the new window starts. For division, the script floors `A` at the symbol's minimum tick. It also calculates a weekday index `h` from the previous window's date using Zeller's congruence, then sets `z = h / 6` (`h` ranges from 0 to 6).

For each multiplier `m = 0, 0.25, 0.50, ... , 21`:

```text
Up level   = C + m × (Lower shadow / A + z)
Down level = C - m × (Upper shadow / A + z)
```

The highlighted multiplier is the ATR rounded **up** to a 0.25 step, capped at 21. Its two target prices are rounded to the symbol's tick size. The level table also shows `DR`, the digital root of the *integer part* of each level's absolute price.

### How the previous window moved

The path study follows the 1-minute prices within a completed window. It measures net change `N` from the first open to the final close, and total travel `T`: the absolute first open-to-close move plus the absolute changes between later closes.

```text
Efficiency ratio = min(1, |N| / T)
Inefficiency (%) = 100 × (1 - efficiency ratio)
```

A high efficiency ratio means the window finished farther from its start relative to the distance traveled. A low ratio means more movement was undone, even if the final close was higher or lower.

The script also measures how far price moved above and below the window open, divides those excursions by ATR, and measures the fraction given back by the close:

```text
Up giveback   = (window high - final close) / (window high - first open)
Down giveback = (final close - window low) / (first open - window low)
```

These fractions are limited to 0–1; a side with no excursion has no giveback value. By default, a **rejected move** needs an excursion of at least 2 ATR and a giveback of at least 60%. A **choppy** path needs efficiency of 0.35 or less and at least four direction changes. The indicator checks choppiness first, then rejected moves, then other low- or high-efficiency paths. These thresholds can be changed in settings.

### Historical next-window comparison

For each completed path category, GoldenShadow records the direction and net change of the *following completed window*. It displays the number of matching examples, next-window up/down rates, the overall up/down baseline, and the average next-window change. Flat outcomes remain in the sample but are neither up nor down, so the displayed up and down rates need not add to 100%.

The panel says history **leans UP** or **leans DOWN** only when there are at least 30 examples, that direction exceeds 50% and the opposite direction, and its rate exceeds the corresponding baseline by at least 5 percentage points. Otherwise it reports insufficient examples or no observed edge. The currently open window is never included as a completed outcome.

## Use

1. Open [`GoldenShadow.pine`](GoldenShadow.pine) in TradingView's Pine Editor and add it to a **1-minute chart** for all features.
2. Select **Fast (14m)** or **Slow (42m)** in *Time Window*.
3. Set the display time zone and adjust the path thresholds or history limit if needed.

The selected time zone affects displayed dates and the weekday calculation; the instrument's bar times define window boundaries. Results depend on the history loaded on the chart. The date factor, digital root, and historical tendencies are descriptive features, not verified forecasts.
