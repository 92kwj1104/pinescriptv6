# Functions - Strategy

Practical Pine Script v6 strategy reference for LLM-assisted code generation.

Use this file when the user asks for backtesting, entries, exits, stop loss, take profit, position sizing, win rate, trade statistics, or strategy conversion.

## Core declaration

A strategy script must use `strategy()` instead of `indicator()`.

```pine
//@version=6
strategy("Example Strategy", overlay = true, initial_capital = 10000, pyramiding = 0)
```

Common declaration options:

| Option | Purpose |
|---|---|
| `overlay = true` | Draw orders and plots on price chart. |
| `initial_capital` | Backtest starting capital. |
| `pyramiding` | Number of additional entries allowed in same direction. Use `0` or `1` for strict single-position logic. |
| `commission_type`, `commission_value` | Model exchange fees. |
| `slippage` | Model execution slippage in ticks. |
| `process_orders_on_close` | Controls whether orders are processed on bar close. |
| `calc_on_every_tick` | Recalculate on realtime ticks. Use carefully because historical behavior may differ. |

## Minimal long strategy

```pine
//@version=6
strategy("EMA Cross Strategy", overlay = true, initial_capital = 10000, pyramiding = 0)

fastLen = input.int(20, "Fast EMA", minval = 1)
slowLen = input.int(50, "Slow EMA", minval = 1)

fast = ta.ema(close, fastLen)
slow = ta.ema(close, slowLen)

longCondition = ta.crossover(fast, slow)
exitCondition = ta.crossunder(fast, slow)

if longCondition
    strategy.entry("L", strategy.long)

if exitCondition
    strategy.close("L")

plot(fast, "Fast EMA")
plot(slow, "Slow EMA")
```

## strategy.entry()

Places an entry order.

Typical usage:

```pine
strategy.entry("L", strategy.long)
strategy.entry("S", strategy.short)
```

With quantity:

```pine
strategy.entry("L", strategy.long, qty = 1)
```

Guidelines:

- Use stable IDs such as `"L"`, `"S"`, `"Long"`, `"Short"`.
- Reuse the same ID when managing one logical position.
- Use `pyramiding = 0` or `1` when the strategy should avoid repeated stacking.
- Do not call long and short entries on the same bar unless the reversal behavior is intentional.

## strategy.close()

Closes an open entry by ID at market.

```pine
if exitCondition
    strategy.close("L")
```

Use this for condition-based exits, such as RSI turning down, trend filter breaking, or opposite signal.

## strategy.exit()

Attaches stop loss and/or take profit orders to an entry.

```pine
strategy.exit("L Exit", from_entry = "L", stop = stopPrice, limit = targetPrice)
```

ATR-based example:

```pine
atr = ta.atr(14)
longStop = strategy.position_avg_price - atr * 1.5
longTarget = strategy.position_avg_price + atr * 3.0

if strategy.position_size > 0
    strategy.exit("L TP/SL", from_entry = "L", stop = longStop, limit = longTarget)
```

Important:

- `stop` is a price level, not a percent.
- `limit` is a price level, not a percent.
- For long positions, stop is usually below entry and limit is above entry.
- For short positions, stop is usually above entry and limit is below entry.
- Keep `strategy.exit()` active while the position exists so the protective order is maintained.

## Percent-based stop and target

```pine
stopPct = input.float(1.0, "Stop %", minval = 0.1) / 100.0
targetPct = input.float(2.0, "Target %", minval = 0.1) / 100.0

longStop = strategy.position_avg_price * (1 - stopPct)
longTarget = strategy.position_avg_price * (1 + targetPct)

if strategy.position_size > 0
    strategy.exit("L TP/SL", from_entry = "L", stop = longStop, limit = longTarget)
```

## Position state variables

| Variable | Meaning |
|---|---|
| `strategy.position_size` | Current position size. Positive for long, negative for short, zero when flat. |
| `strategy.position_avg_price` | Average entry price of current position. |
| `strategy.equity` | Current equity in backtest. |
| `strategy.openprofit` | Unrealized profit/loss. |
| `strategy.netprofit` | Closed net profit. |
| `strategy.closedtrades` | Number of closed trades. |
| `strategy.opentrades` | Number of currently open trades. |
| `strategy.wintrades` | Number of winning closed trades. |
| `strategy.losstrades` | Number of losing closed trades. |

Win-rate table snippet:

```pine
winRate = strategy.closedtrades > 0 ? strategy.wintrades / strategy.closedtrades * 100 : na
```

## Long and short skeleton

```pine
//@version=6
strategy("Long Short Skeleton", overlay = true, pyramiding = 0)

emaFast = ta.ema(close, 20)
emaSlow = ta.ema(close, 50)
atr = ta.atr(14)

longSignal = ta.crossover(emaFast, emaSlow)
shortSignal = ta.crossunder(emaFast, emaSlow)

if longSignal and strategy.position_size <= 0
    strategy.entry("L", strategy.long)

if shortSignal and strategy.position_size >= 0
    strategy.entry("S", strategy.short)

if strategy.position_size > 0
    strategy.exit("L Exit", "L", stop = strategy.position_avg_price - atr * 1.5, limit = strategy.position_avg_price + atr * 3.0)

if strategy.position_size < 0
    strategy.exit("S Exit", "S", stop = strategy.position_avg_price + atr * 1.5, limit = strategy.position_avg_price - atr * 3.0)
```

## Entry filtering checklist

For scalping strategies, avoid relying on one signal only. Combine filters deliberately.

Common filters:

- Trend: EMA alignment, Supertrend direction, VWAP bias.
- Momentum: RSI, Stochastic, MACD histogram.
- Volatility: ATR threshold, Bollinger Band width, Keltner width.
- Volume: volume above moving average, volume spike.
- Market structure: recent high breakout, pullback reclaim, pivot level break.
- Session/time filter: avoid low-liquidity hours.
- Higher timeframe filter: use `request.security()` with non-repainting logic.

## Repainting and backtest safety

Rules for LLMs:

1. Do not use future-looking values.
2. Do not use `barmerge.lookahead_on` unless the expression is intentionally offset and the repainting implications are explained.
3. Prefer confirmed-bar signals for alerts and automation:

```pine
confirmedLong = longCondition and barstate.isconfirmed
```

4. If using pivots, remember pivot signals appear only after right-side bars have elapsed.
5. Backtest results are not live trading guarantees. Add fees, slippage, and realistic exits.

## Strategy-to-indicator conversion

When the user wants an indicator instead of a backtest:

- Replace `strategy()` with `indicator()`.
- Replace `strategy.entry()` with `plotshape()` or `label.new()`.
- Replace `strategy.exit()` with plotted stop/target lines.
- Add `alertcondition()` for buy/sell/exit signals.

Example:

```pine
alertcondition(longCondition, "Long Signal", "Long signal detected")
```

## Common mistakes

| Mistake | Fix |
|---|---|
| Passing percent to `stop` or `limit` | Convert percent to actual price first. |
| Calling `strategy.exit()` only on entry bar | Keep it active while position exists. |
| Too many entries on consecutive bars | Use `strategy.position_size == 0` or pyramiding control. |
| Unrealistic win rate | Add fees, slippage, no-repaint HTF logic, and confirmed-bar rules. |
| Mixing indicator and strategy declarations | Use only one declaration: `indicator()` or `strategy()`. |
| Using labels/tables inside `request.security()` expression | Keep drawings outside request expressions. |

## Recommended strategy prompt for LLMs

When generating strategy code:

1. Start with `//@version=6`.
2. Use `strategy()`.
3. Define inputs first.
4. Calculate indicators.
5. Build explicit long/short conditions.
6. Add position guards.
7. Add stop loss and take profit using actual price levels.
8. Add plots for visual inspection.
9. Add a small performance table only if requested.
10. Mention repainting assumptions clearly.
