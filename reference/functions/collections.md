# Functions - Collections: Arrays, Maps, Matrices

Practical Pine Script v6 collections reference for LLM-assisted code generation.

Use this file when the user asks for arrays, maps, matrices, rolling lists, object management, custom dashboards, storing previous signals, deleting old labels/lines/boxes, or lower-timeframe arrays from `request.security_lower_tf()`.

## Collection types

Pine Script supports collection objects such as:

| Type | Use |
|---|---|
| `array<T>` | Ordered list of values or object IDs. Most common collection type. |
| `map<K, V>` | Key-value storage. Useful for symbol/state lookup. |
| `matrix<T>` | Two-dimensional numeric or generic table-like storage. Less common in indicators. |

Collections are object IDs and are generally series values. Use `var` when the collection should persist across bars.

## Arrays

### Create arrays

```pine
array<float> values = array.new<float>()
array<float> fixedValues = array.new<float>(10, na)
arr = array.from(open, high, low, close)
```

Persistent array:

```pine
var array<float> closes = array.new<float>()
```

### Push and read values

```pine
array.push(closes, close)
lastValue = array.get(closes, array.size(closes) - 1)
```

### Set values

```pine
array.set(closes, 0, close)
```

### Size

```pine
count = array.size(closes)
```

### Pop, shift, clear

```pine
last = array.pop(closes)
first = array.shift(closes)
array.clear(closes)
```

Use `array.shift()` to remove the oldest value from a rolling window.

## Rolling array pattern

Stores the latest N values.

```pine
//@version=6
indicator("Rolling Array", overlay = false)

len = input.int(20, "Stored values", minval = 1)
var array<float> values = array.new<float>()

array.push(values, close)
if array.size(values) > len
    array.shift(values)

sum = 0.0
for i = 0 to array.size(values) - 1
    sum += array.get(values, i)

avg = array.size(values) > 0 ? sum / array.size(values) : na
plot(avg)
```

Guidelines:

- Guard loops when arrays may be empty.
- Keep arrays small for performance.
- Prefer built-in `ta.*` functions for common rolling calculations.
- Use arrays when storing objects, custom states, or non-standard calculations.

## Arrays of object IDs

Useful for deleting old labels, lines, or boxes.

```pine
//@version=6
indicator("Label Array", overlay = true, max_labels_count = 200)

var array<label> labels = array.new<label>()

longSignal = ta.crossover(ta.ema(close, 20), ta.ema(close, 50))

if longSignal
    newLabel = label.new(bar_index, low, "BUY", style = label.style_label_up)
    array.push(labels, newLabel)

maxLabels = 50
if array.size(labels) > maxLabels
    oldLabel = array.shift(labels)
    label.delete(oldLabel)
```

Line array:

```pine
var array<line> lines = array.new<line>()

if barstate.islastconfirmedhistory
    newLine = line.new(bar_index - 10, high, bar_index, high)
    array.push(lines, newLine)

if array.size(lines) > 20
    oldLine = array.shift(lines)
    line.delete(oldLine)
```

Box array:

```pine
var array<box> zones = array.new<box>()

newBox = box.new(bar_index - 5, high, bar_index, low)
array.push(zones, newBox)

if array.size(zones) > 10
    oldBox = array.shift(zones)
    box.delete(oldBox)
```

## Looping arrays

```pine
sum = 0.0
for i = 0 to array.size(values) - 1
    sum += array.get(values, i)
```

Safe pattern:

```pine
sum = 0.0
if array.size(values) > 0
    for i = 0 to array.size(values) - 1
        sum += array.get(values, i)
```

Avoid expensive nested loops. Pine has execution limits.

## request.security_lower_tf() arrays

`request.security_lower_tf()` returns an array containing lower-timeframe values inside the current chart bar.

Example pattern:

```pine
//@version=6
indicator("Lower TF Array", overlay = false)

ltf = input.timeframe("1", "Lower timeframe")
arr = request.security_lower_tf(syminfo.tickerid, ltf, close)

float ltfSum = 0.0
if array.size(arr) > 0
    for i = 0 to array.size(arr) - 1
        ltfSum += array.get(arr, i)

ltfAvg = array.size(arr) > 0 ? ltfSum / array.size(arr) : na
plot(ltfAvg)
```

Guidelines:

- Always check `array.size(arr) > 0` before looping.
- Lower-timeframe arrays can become large. Keep calculations simple.
- Do not draw labels/lines/boxes inside lower-timeframe request expressions.

## Maps

Maps store key-value pairs.

```pine
var map<string, float> levels = map.new<string, float>()

map.put(levels, "support", low)
map.put(levels, "resistance", high)

support = map.get(levels, "support")
```

Map with symbol states:

```pine
var map<string, string> states = map.new<string, string>()
map.put(states, syminfo.ticker, close > ta.ema(close, 50) ? "UP" : "DOWN")
currentState = map.get(states, syminfo.ticker)
```

Guidelines:

- Use maps for named state lookup.
- Use arrays for ordered lists.
- Keep map keys simple: strings, enums, or stable identifiers.

## Matrices

Matrices are two-dimensional collections.

```pine
matrix<float> m = matrix.new<float>(2, 2, 0.0)
matrix.set(m, 0, 0, 1.0)
value = matrix.get(m, 0, 0)
```

Use cases:

- Small numeric grids.
- Custom scoring models.
- Correlation-like layouts.

For most TradingView indicators, arrays and tables are more practical than matrices.

## Collection-backed dashboard pattern

Use arrays to keep row labels and values, then render them into a table.

```pine
//@version=6
indicator("Array Dashboard", overlay = true)

rsi = ta.rsi(close, 14)
ema = ta.ema(close, 50)
trend = close > ema ? "UP" : "DOWN"

var table dash = table.new(position.top_right, 2, 3)
var array<string> names = array.from("RSI", "Trend", "Close")

if barstate.islast
    values = array.from(str.tostring(rsi, "#.0"), trend, str.tostring(close, "#.00"))
    for i = 0 to array.size(names) - 1
        table.cell(dash, 0, i, array.get(names, i))
        table.cell(dash, 1, i, array.get(values, i))
```

## Persistent state with var

Without `var`, collections are recreated on each bar.

Correct:

```pine
var array<float> savedSignals = array.new<float>()
```

Usually wrong for persistent state:

```pine
array<float> savedSignals = array.new<float>()
```

## Common mistakes

| Mistake | Fix |
|---|---|
| Reading an empty array | Check `array.size(arr) > 0`. |
| Using `array.get(arr, array.size(arr))` | Last index is `array.size(arr) - 1`. |
| Forgetting `var` for persistent arrays | Use `var array<T> name = array.new<T>()`. |
| Creating endless labels/lines/boxes | Store IDs in arrays and delete old objects. |
| Using arrays for simple moving averages | Prefer `ta.sma`, `ta.ema`, etc. |
| Heavy nested loops | Simplify to avoid Pine execution limits. |

## LLM generation checklist

When generating collection-heavy Pine code:

1. Declare persistent collections with `var`.
2. Guard all array reads with size checks.
3. Delete old visual objects if storing many IDs.
4. Keep loops short.
5. Prefer built-in `ta.*` functions for standard indicators.
6. Use tables for display, arrays for storage, maps for named lookup.
