# Day Trading Strategy — TradingView Indicator

Pine Script v5 indicator implementing a levels + trend + patience candle strategy for intraday stock trading.

## Strategy Overview

### Timeframe Rules
| Time (ET)       | Chart Timeframe |
|-----------------|-----------------|
| 9:30 – 10:00   | 2-minute        |
| 10:00 – 11:00  | 5-minute        |
| 11:00 – Close  | 10-minute       |

### Three Components

**1. Levels**
- Drawn manually before market open on the hourly chart
- Key areas: yesterday's high/low, psychological levels, SPX $50 increments, areas of repeated rejection/support
- Never trade inside the pre-market range (PMH–PML)
- All levels = solid yellow lines

**2. Trend**
- Wait for price to break out of the pre-market range (above PMH or below PML)
- Trade direction follows breakout direction

**3. Patience Candle (PC)**
- After trend + breakout: wait for a pullback
- **Long:** pullback candle high ≤ recent swing high (candle contained to upside)
- **Short:** pullback candle low ≥ recent swing low (candle contained to downside)
- Each new candle in the pullback becomes the new PC candidate
- **Entry:** price overtakes PC high (long) or undercuts PC low (short)
- **Stop:** PC low (long) or PC high (short)
- **Invalidation:** if price hits 0.5 Fibonacci retracement of the move → wait for new swing

### R:R Filter
- PC range (high–low) must be ≤ distance to next key level
- Minimum R:R configurable (default 1.0R)

### Always-On Indicators
- VWAP (current session)
- 8 SMA, 21 SMA (current timeframe)
- 5 SMA, 10 SMA (always shown)
- Hourly 200 SMA (always shown, used for confluence and as trade barrier)

## Files

| File | Description |
|------|-------------|
| `strategy_v1.pine` | v1 indicator — core logic |

## How to Use in TradingView

1. Open TradingView → Pine Script Editor (bottom panel)
2. Click **New** → paste the contents of `strategy_v1.pine`
3. Click **Add to chart**
4. Set up alerts via the Alerts panel using the built-in alert conditions

## Versions

- **v1** — Core indicator: VWAP, SMAs, PM range, patience candle detection, entry/stop levels, fib invalidation, alerts
