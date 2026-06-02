# Functions - General, Inputs, Math, Strings, Time, Alerts

Practical Pine Script v6 general reference for LLM-assisted code generation.

Use this file when the user asks about inputs, math helpers, string formatting, time/session filters, alert messages, runtime errors, or general utility logic.

## Script declarations

Use one declaration per script:

```pine
//@version=6
indicator("My Indicator", overlay = true)
```

or:

```pine
//@version=6
strategy("My Strategy", overlay = true)
```

Rules:

- Do not use both `indicator()` and `strategy()` in the same script.
- Always start generated code with `//@version=6`.
- Use `overlay = true` for price-chart indicators.
- Use a separate pane only for oscillators unless the user asks otherwise.

## input.*()

Inputs allow users to configure the script from TradingView settings.

Common inputs:

```pine
len = input.int(14, "Length", minval = 1)
src = input.source(close, "Source")
mult = input.float(2.0, "Multiplier", minval = 0.1, step = 0.1)
showSignals = input.bool(true, "Show signals")
tf = input.timeframe("60", "Higher timeframe")
sessionInput = input.session("0900-1600", "Session")
mode = input.string("Balanced", "Mode", options = ["Aggressive", "Balanced", "Strict"])
```

Guidelines:

- Put inputs near the top of the script.
- Use `minval`, `maxval`, and `step` where appropriate.
- Use clear titles because users see these in the UI.
- For presets, use `input.string(..., options = [...])`.

## math.*

Common math functions:

| Function | Use |
|---|---|
| `math.abs(x)` | Absolute value. |
| `math.max(a, b)` | Larger value. |
| `math.min(a, b)` | Smaller value. |
| `math.round(x)` | Round to nearest integer. |
| `math.floor(x)` | Round down. |
| `math.ceil(x)` | Round up. |
| `math.pow(x, y)` | Power. |
| `math.sqrt(x)` | Square root. |

Examples:

```pine
atr = ta.atr(14)
rangePct = math.abs(close - open) / close * 100
stop = close - atr * 1.5
target = close + atr * 3.0
```

Clamp helper:

```pine
clamp(x, lo, hi) =>
    math.max(lo, math.min(x, hi))
```

## nz(), na(), fixnan()

Use these helpers for missing values.

```pine
safeClose = nz(close, close[1])
isMissing = na(close)
```

Common patterns:

```pine
prev = nz(mySeries[1], mySeries)
validSignal = not na(rsi) and rsi > 50
```

Guidelines:

- Use `na(x)` to test missing values.
- Do not compare directly with `x == na`.
- Use `nz()` to replace missing values only when that replacement makes logical sense.

## str.*

String helpers are useful for labels, tables, and alerts.

Common functions:

```pine
str.tostring(close)
str.tostring(close, "#.00")
str.format("RSI: {0}, Close: {1}", rsi, close)
str.contains(syminfo.ticker, "BTC")
```

Table text example:

```pine
rsiText = str.tostring(ta.rsi(close, 14), "#.0")
priceText = str.tostring(close, "#.00")
```

Alert message example:

```pine
msg = "BUY " + syminfo.ticker + " close=" + str.tostring(close)
```

## color helpers

Use `color.new()` for transparency.

```pine
softGreen = color.new(color.green, 85)
softRed = color.new(color.red, 85)
bgcolor(trendUp ? softGreen : na)
```

Transparency ranges from 0 to 100:

- `0` means fully opaque.
- `100` means invisible.
- For backgrounds and zones, 85-95 is usually readable.

## time and session helpers

Use time filters to avoid unwanted trading periods.

Session filter:

```pine
sessionInput = input.session("0900-1600", "Trading Session")
inSession = not na(time(timeframe.period, sessionInput))
```

Date range filter:

```pine
startTime = input.time(timestamp("2024-01-01T00:00:00"), "Start")
endTime = input.time(timestamp("2026-12-31T23:59:59"), "End")
inDateRange = time >= startTime and time <= endTime
```

Higher timeframe change detection:

```pine
isNewDay = ta.change(time("1D")) != 0
```

Timeframe conversion:

```pine
chartMinutes = timeframe.in_seconds() / 60
inputMinutes = timeframe.in_seconds("60") / 60
```

Important timeframe rule:

- Pine uses minutes without an hour suffix.
- Use `"60"` for one hour, not `"1H"`.

## barstate.*

Useful bar state variables:

| Variable | Use |
|---|---|
| `barstate.isfirst` | Initialize once on first bar. |
| `barstate.islast` | Update tables/labels only on last visible calculation. |
| `barstate.isconfirmed` | True when bar is closed/confirmed. |
| `barstate.isrealtime` | True on realtime updates. |
| `barstate.islastconfirmedhistory` | Last confirmed historical bar. |

Confirmed signal:

```pine
confirmedLong = longSignal and barstate.isconfirmed
```

Efficient table update:

```pine
if barstate.islast
    table.cell(dash, 0, 0, "Status")
```

## alertcondition()

For user-selectable TradingView alerts.

```pine
alertcondition(longSignal, "BUY Signal", "BUY on {{ticker}} {{interval}}")
alertcondition(shortSignal, "SELL Signal", "SELL on {{ticker}} {{interval}}")
```

Confirmed alerts:

```pine
alertcondition(longSignal and barstate.isconfirmed, "Confirmed BUY", "Confirmed BUY on {{ticker}} {{interval}}")
```

## alert()

For dynamic alert messages.

```pine
if longSignal and barstate.isconfirmed
    alert("BUY " + syminfo.ticker + " close=" + str.tostring(close), alert.freq_once_per_bar_close)
```

Common alert frequencies:

| Frequency | Meaning |
|---|---|
| `alert.freq_all` | Every call can trigger. |
| `alert.freq_once_per_bar` | First call per bar. |
| `alert.freq_once_per_bar_close` | Only when the realtime bar closes. |

## runtime.error()

Use to stop script execution when user inputs are invalid.

```pine
if timeframe.in_seconds() > timeframe.in_seconds("60")
    runtime.error("Use this script on 60-minute or lower timeframes.")
```

Use sparingly. Prefer graceful behavior when possible.

## User-defined functions

Use functions to keep code short and avoid huge `if` blocks.

```pine
isBullTrend(src, len) =>
    src > ta.ema(src, len)

riskLevels(entry, atrValue, stopMult, targetMult) =>
    [entry - atrValue * stopMult, entry + atrValue * targetMult]
```

Tuple return example:

```pine
calcBands(src, len, mult) =>
    basis = ta.sma(src, len)
    dev = ta.stdev(src, len) * mult
    [basis, basis + dev, basis - dev]

[basis, upper, lower] = calcBands(close, 20, 2.0)
```

## Conditional expressions

Ternary operator:

```pine
state = fast > slow ? "UP" : "DOWN"
level = strategy.position_size > 0 ? stopPrice : na
```

Use nested ternaries carefully. If readability drops, use `if` or helper functions.

## Common general template

```pine
//@version=6
indicator("General Template", overlay = true)

len = input.int(20, "EMA Length", minval = 1)
showSignals = input.bool(true, "Show Signals")

ema = ta.ema(close, len)
trendUp = close > ema
longSignal = ta.crossover(close, ema) and barstate.isconfirmed

plot(ema, "EMA")
plotshape(showSignals and longSignal, "BUY", shape.labelup, location.belowbar, text = "BUY")
alertcondition(longSignal, "BUY", "BUY on {{ticker}} {{interval}}")
```

## Common mistakes

| Mistake | Fix |
|---|---|
| Comparing with `na` using `==` | Use `na(x)`. |
| Using `"1H"` timeframe | Use `"60"`. |
| Updating tables on every bar | Use `barstate.islast`. |
| Overusing dynamic labels | Prefer `plotshape()` or table. |
| Huge repeated logic | Move repeated logic into functions. |
| Intrabar repaint confusion | Use `barstate.isconfirmed` for confirmed signals. |
