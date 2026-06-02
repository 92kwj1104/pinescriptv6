# Functions - Drawing and Visuals

Practical Pine Script v6 visual reference for LLM-assisted indicator and strategy generation.

Use this file when the user asks for chart visuals, BUY/SELL labels, stop-loss lines, take-profit lines, support/resistance boxes, background highlights, tables, mobile-friendly dashboards, or alerts.

## Visual design principles

For trading indicators:

1. Keep the chart readable.
2. Use shapes for rare event signals.
3. Use plots for continuous values.
4. Use labels sparingly because they can hit object limits.
5. Use tables for compact status dashboards.
6. Avoid creating new lines/labels/boxes every bar unless necessary.
7. Prefer updating persistent objects with `var` and setter functions.

## plot()

Plots a continuous series.

```pine
//@version=6
indicator("Plot Example", overlay = true)
ema20 = ta.ema(close, 20)
plot(ema20, "EMA 20")
```

Common options:

```pine
plot(ema20, title = "EMA 20", linewidth = 2)
plot(stopPrice, title = "Stop", style = plot.style_linebr)
```

Useful styles:

| Style | Use |
|---|---|
| `plot.style_line` | Normal line. |
| `plot.style_linebr` | Breaks line on `na`; useful for active stop/target only during positions. |
| `plot.style_columns` | Histogram/volume-style columns. |
| `plot.style_stepline` | Step-like levels. |
| `plot.style_cross` | Cross markers. |

Active position line example:

```pine
activeStop = strategy.position_size != 0 ? stopPrice : na
plot(activeStop, "Active Stop", style = plot.style_linebr)
```

## hline()

Draws a fixed horizontal level, mostly for oscillator panes.

```pine
rsi = ta.rsi(close, 14)
plot(rsi, "RSI")
hline(70, "Overbought")
hline(30, "Oversold")
```

## plotshape()

Draws signal icons on bars. Good for BUY/SELL signals.

```pine
longSignal = ta.crossover(ta.ema(close, 20), ta.ema(close, 50))
plotshape(longSignal, title = "BUY", style = shape.triangleup, location = location.belowbar, text = "BUY", size = size.small)
```

Common shapes:

| Shape | Typical use |
|---|---|
| `shape.triangleup` | Buy signal. |
| `shape.triangledown` | Sell signal. |
| `shape.circle` | State marker. |
| `shape.cross` | Invalid/exit marker. |
| `shape.labelup` | Text label below bar. |
| `shape.labeldown` | Text label above bar. |

Common locations:

| Location | Meaning |
|---|---|
| `location.abovebar` | Above candle. |
| `location.belowbar` | Below candle. |
| `location.top` | Top of pane. |
| `location.bottom` | Bottom of pane. |
| `location.absolute` | Uses explicit y value. |

## plotchar()

Draws a character on bars. Useful for lightweight debugging.

```pine
plotchar(longSignal, title = "Long debug", char = "▲", location = location.belowbar)
```

## bgcolor()

Colors the chart or pane background when a condition is true.

```pine
trendUp = ta.ema(close, 20) > ta.ema(close, 50)
bgcolor(trendUp ? color.new(color.green, 90) : na)
```

Use high transparency values such as 85-95 for subtle backgrounds.

## barcolor()

Colors price bars from a script.

```pine
barcolor(close > open ? color.green : color.red)
```

Use carefully. Too much candle coloring can reduce readability.

## fill()

Fills the area between two plot objects.

```pine
basis = ta.sma(close, 20)
dev = ta.stdev(close, 20) * 2
upperPlot = plot(basis + dev, "Upper")
lowerPlot = plot(basis - dev, "Lower")
fill(upperPlot, lowerPlot, color.new(color.blue, 90))
```

## label.new()

Creates a text label at a bar or price location.

```pine
if longSignal
    label.new(bar_index, low, "BUY", style = label.style_label_up, textcolor = color.white)
```

Guidelines:

- Labels consume object limits.
- Use `max_labels_count` in `indicator()` or `strategy()` if needed.
- Avoid creating labels every bar.
- For live dashboards, prefer `table` instead of many labels.

Persistent label pattern:

```pine
//@version=6
indicator("Persistent Label", overlay = true, max_labels_count = 50)

var label statusLabel = na
if barstate.islast
    if na(statusLabel)
        statusLabel := label.new(bar_index, close, "Status")
    label.set_x(statusLabel, bar_index)
    label.set_y(statusLabel, close)
    label.set_text(statusLabel, "Close: " + str.tostring(close))
```

Useful label setters:

```pine
label.set_text(myLabel, "New text")
label.set_x(myLabel, bar_index)
label.set_y(myLabel, close)
label.set_color(myLabel, color.green)
label.set_textcolor(myLabel, color.white)
label.delete(myLabel)
```

## line.new()

Creates a line object.

```pine
var line stopLine = na
if barstate.islast
    if na(stopLine)
        stopLine := line.new(bar_index - 10, close, bar_index, close)
    line.set_xy1(stopLine, bar_index - 10, close)
    line.set_xy2(stopLine, bar_index, close)
```

Use cases:

- Stop-loss line.
- Take-profit line.
- Trendline.
- Previous high/low level.
- Session open line.

Common setters:

```pine
line.set_xy1(myLine, x1, y1)
line.set_xy2(myLine, x2, y2)
line.set_color(myLine, color.red)
line.set_width(myLine, 2)
line.delete(myLine)
```

Stop/target line pattern:

```pine
longActive = strategy.position_size > 0
stopPrice = strategy.position_avg_price * 0.99
targetPrice = strategy.position_avg_price * 1.02

plot(longActive ? stopPrice : na, "Stop", style = plot.style_linebr)
plot(longActive ? targetPrice : na, "Target", style = plot.style_linebr)
```

For simple horizontal levels, prefer `plot(..., style = plot.style_linebr)` over many `line.new()` objects.

## box.new()

Creates a rectangle object. Useful for support/resistance zones, supply/demand zones, fair value gaps, and consolidation ranges.

```pine
//@version=6
indicator("Box Example", overlay = true, max_boxes_count = 100)

rangeHigh = ta.highest(high, 20)
rangeLow = ta.lowest(low, 20)

var box rangeBox = na
if barstate.islast
    if na(rangeBox)
        rangeBox := box.new(bar_index - 20, rangeHigh, bar_index, rangeLow)
    box.set_left(rangeBox, bar_index - 20)
    box.set_right(rangeBox, bar_index)
    box.set_top(rangeBox, rangeHigh)
    box.set_bottom(rangeBox, rangeLow)
```

Useful box setters:

```pine
box.set_left(myBox, bar_index - 20)
box.set_right(myBox, bar_index)
box.set_top(myBox, high)
box.set_bottom(myBox, low)
box.set_bgcolor(myBox, color.new(color.gray, 90))
box.set_border_color(myBox, color.gray)
box.delete(myBox)
```

## table.new()

Creates a fixed-position table. Best for mobile-friendly status panels.

```pine
//@version=6
indicator("Mobile Dashboard", overlay = true)

rsi = ta.rsi(close, 14)
trendUp = close > ta.ema(close, 50)

var table dash = table.new(position.top_right, 2, 3)

if barstate.islast
    table.cell(dash, 0, 0, "Metric")
    table.cell(dash, 1, 0, "Value")
    table.cell(dash, 0, 1, "RSI")
    table.cell(dash, 1, 1, str.tostring(rsi, "#.0"))
    table.cell(dash, 0, 2, "Trend")
    table.cell(dash, 1, 2, trendUp ? "UP" : "DOWN")
```

Guidelines:

- Declare tables once using `var table`.
- Update tables only on `barstate.islast` for performance.
- Keep mobile tables small: 2 columns and 3-6 rows is usually enough.
- Avoid giant dashboards that cover the chart.

Common positions:

| Position | Use |
|---|---|
| `position.top_right` | Default dashboard. |
| `position.bottom_right` | Less intrusive dashboard. |
| `position.top_left` | Alternative if price scale is crowded. |
| `position.middle_right` | Debug panel. |

## alertcondition()

Creates selectable alert conditions in TradingView.

```pine
alertcondition(longSignal, title = "BUY Signal", message = "BUY signal on {{ticker}} {{interval}}")
alertcondition(shortSignal, title = "SELL Signal", message = "SELL signal on {{ticker}} {{interval}}")
```

Rules:

- Use `alertcondition()` for user-configurable alerts.
- Use confirmed conditions if avoiding intrabar repainting:

```pine
confirmedLong = longSignal and barstate.isconfirmed
alertcondition(confirmedLong, "Confirmed BUY", "Confirmed BUY on {{ticker}}")
```

## alert()

Triggers dynamic alert messages from code when used with TradingView alert setup.

```pine
if longSignal and barstate.isconfirmed
    alert("BUY " + syminfo.ticker + " close=" + str.tostring(close), alert.freq_once_per_bar_close)
```

Use `alertcondition()` unless dynamic messages are needed.

## Mobile-friendly signal template

```pine
//@version=6
indicator("Mobile Signal Template", overlay = true, max_labels_count = 100)

fast = ta.ema(close, 20)
slow = ta.ema(close, 50)
rsi = ta.rsi(close, 14)

longSignal = ta.crossover(fast, slow) and rsi > 50
shortSignal = ta.crossunder(fast, slow) and rsi < 50

plot(fast, "Fast EMA")
plot(slow, "Slow EMA")
plotshape(longSignal, "BUY", shape.labelup, location.belowbar, text = "BUY", size = size.small)
plotshape(shortSignal, "SELL", shape.labeldown, location.abovebar, text = "SELL", size = size.small)

var table dash = table.new(position.top_right, 2, 4)
if barstate.islast
    table.cell(dash, 0, 0, "State")
    table.cell(dash, 1, 0, longSignal ? "BUY" : shortSignal ? "SELL" : "WAIT")
    table.cell(dash, 0, 1, "RSI")
    table.cell(dash, 1, 1, str.tostring(rsi, "#.0"))
    table.cell(dash, 0, 2, "Trend")
    table.cell(dash, 1, 2, fast > slow ? "UP" : "DOWN")
    table.cell(dash, 0, 3, "TF")
    table.cell(dash, 1, 3, timeframe.period)

alertcondition(longSignal and barstate.isconfirmed, "BUY", "BUY on {{ticker}} {{interval}}")
alertcondition(shortSignal and barstate.isconfirmed, "SELL", "SELL on {{ticker}} {{interval}}")
```

## Object limit checklist

When generating visual scripts:

- Set `max_labels_count`, `max_lines_count`, and `max_boxes_count` only when needed.
- Use `var` objects and setters for continuously updated visuals.
- Use `plotshape()` for repeated simple signals instead of endless `label.new()`.
- Use `plot()` for stop/target levels when possible.
- Delete old objects if creating many objects.

## Common mistakes

| Mistake | Fix |
|---|---|
| Creating a new label every bar | Use `plotshape()` or persistent `label` updated with setters. |
| Updating table on every historical bar | Wrap table updates in `if barstate.islast`. |
| Drawing too many boxes | Limit creation or delete old boxes. |
| Using opaque backgrounds | Use `color.new(color, 85-95)` transparency. |
| Mixing pane and overlay assumptions | Use `overlay = true` for price-chart visuals. |
| Forgetting alerts | Add `alertcondition()` for actionable signals. |
