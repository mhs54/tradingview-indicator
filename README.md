# TradingView Indicator Suite

Pine Script v5/v6 indicators for the **LTP day-trading strategy** (Levels + Trend + Patience Candle) on US equities. The suite overlays on 2m / 5m / 10m intraday charts with session-aware (US/Eastern) logic.

For the strategy framework itself, see `/Users/mitchellsmith/dev/backtest-engine/docs/strategy.md`.

---

## Files

| File | Purpose |
|------|---------|
| `strategy_v1.pine` | Original indicator — manual levels + PC detection + arrows |
| `strategy_v2.pine` | Adds auto-detected Key Levels (7-source priority hierarchy) + push notifications |
| `volume_v1.pine` | Normalized 0–100 volume pane with MA overlay |
| `sr_channels_v1.pine` | Hourly pivot clusters with strength-based channel detection (legacy) |

Also at `/Users/mitchellsmith/code/support_resistance.pine` — simpler hourly pivot lines with merge-zone deduplication.

---

## Strategy v1 — `strategy_v1.pine`

Core indicator for the manual visual strategy.

### Timeframe rules (ET)
| Time | Chart TF |
|------|----------|
| 9:30 – 10:00 | 2m |
| 10:00 – 11:00 | 5m |
| 11:00 – close | 10m |

### Three components
**1. Levels** — drawn manually on the hourly chart (PMH/PML, yesterday H/L, psychological, SPX $50 increments, repeated rejections). Never trade inside the PM range.

**2. Trend** — direction of the PM range breakout (close > PMH → Long, close < PML → Short).

**3. Patience Candle** — after breakout, wait for a pullback candle that:
- Long: high ≤ recent swing high (contained to upside)
- Short: low ≥ recent swing low (contained to downside)

Each new pullback bar becomes the new PC candidate. Entry = break of PC high (Long) / low (Short). Stop = PC low (Long) / high (Short). Invalidation at **0.5 Fib retrace** (the backtest engine evolved this to 40% retrace with latching).

### Visuals
- PMH/PML — white lines, last 5 days
- Opening range — lime green, 9:30–9:45 ET
- VWAP (teal), SMAs 8/21 (blue/purple), 5m/10m/1h 200 SMAs
- Patience candle arrows — green polyline triangles (Long) / red (Short)
- Warning label `!` if hourly 200 SMA is within $1.00 of entry (configurable)

### Alert conditions
- `Patience Candle - LONG` / `SHORT`
- `PM Breakout - LONG` / `SHORT`

### Inputs
Show toggles for each SMA, VWAP, PMH/PML, Opening Range, PC arrows. Warning distance ($1.00 default).

---

## Strategy v2 — `strategy_v2.pine`

v1 + auto-detected Key Levels. This is what the backtest engine's `levels.py` is modeled after.

### Key Levels discovery (priority hierarchy, highest first)
| Priority | Source |
|---|---|
| 60 | All-Time High (2-year RTH daily high) |
| 50 | Hourly 200 SMA |
| 40 | Psychological levels ($50 multiples) within ADR range |
| 30 – 11 | Daily RTH H/L, last 10 days (decreasing priority per day depth) |
| 10.5 – 2.9 | Daily RTH O/C, last 10 days |
| 2.5 – 2.2 | Hourly RTH OHLC from last 30 bars (~8–9 days) |
| 1.5 | Extended-session H/L (PM + AH) where ETH exceeded RTH range |
| 0.5 | Monthly OHLC, 5 prior months |

### Filtering
- Min spacing between levels (`$1.00` default, configurable)
- Range = ADR × `kl_adr_mult` (1.0 default) above PMH and below PML
- Exclude levels within $0.50 of PM boundaries
- Cap at 5 levels above / 5 below (configurable)
- Greedy accept after bubble-sort by priority DESC, then distance to today's open ASC

### Behavior
Drawn once at session open, extends through close, no mid-day updates.

### Added features vs v1
- Push notifications via `alert()` function (toggle) — fires `"{TICKER} {TF}m — LONG/SHORT PC formed..."` on PC confirmation
- Patience candle confirmation requires `barstate.isconfirmed` (no intrabar triggers)
- SMA colors changed: 8 green, 21 red, 5m 200 yellow, 10m 200 orange, 1h 200 hot pink

### Inputs (beyond v1)
- Show Key Levels (bool)
- Min Spacing (`$1.00` default)
- ADR Multiplier (`1.0`)
- Max Levels Per Side (`5`)
- Push Notifications (bool)

### Notable version history (git log)
- Add hourly OHLC levels + confluence-based selection
- Add ATH (priority 60)
- Raise min spacing $0.80 → $1.00
- Extend lookback: daily 7→10 days, hourly 30→60 bars
- Replace fixed range with ADR-based dynamic range

---

## Volume — `volume_v1.pine`

Standalone pane indicator (not overlay). Normalizes volume bars to 0–100 over a 100-bar lookback, overlaid with a 10-bar SMA.

**Inputs:** MA Length (10), Normalization Lookback (100)

**Colors:** green up-bars, red down-bars (30% transparency), yellow MA line.

No alerts.

---

## SR Channels — `sr_channels_v1.pine` (legacy)

Adapted from LonesomeTheBlue's "Support Resistance Channels." Replaced by the Key Levels system in `strategy_v2.pine` — kept for reference.

### Logic
- Detects hourly pivots (`ta.pivothigh` / `ta.pivotlow`)
- Clusters pivots into channels within a width = `ChannelW%` × 300-bar range
- Channel strength = count of bars touching the level (capped 100)
- Accepts channels with strength ≥ `minstrength × 20`
- Displays 3 closest above and 3 closest below current price (yellow lines)

### Inputs
- Pivot Period (10)
- Source (High/Low or Close/Open)
- Max Channel Width % (5)
- Minimum Strength (1)
- Loopback Period Days (60)

### Alerts
- `Resistance Broken` — close crossed above a resistance line
- `Support Broken` — close crossed below a support line

---

## Simple S/R — `/Users/mitchellsmith/code/support_resistance.pine`

Lightweight alternative to `sr_channels_v1`. Hourly pivots drawn as red (resistance) / green (support) lines with optional price labels. Merges nearby levels by % proximity. FIFO cap at `maxLevels` (20 default). No clustering, no strength calc.

**Inputs:** Pivot Left/Right bars (10/10), Line Width, Max Levels, Extend Lines Right, Merge Zone % (0.1)

No alerts.

---

## How to use in TradingView

1. Open TradingView → Pine Script Editor (bottom panel)
2. Click **New** → paste file contents
3. Click **Add to chart**
4. Set alerts via the Alerts panel using the built-in alert conditions, or enable Push Notifications in v2 inputs and create a single "Any alert() function call" alert

For the full strategy framework (what LTP means, why the filters exist, how the backtest engine evolved from this v1 spec), see:
- `/Users/mitchellsmith/dev/backtest-engine/docs/strategy.md`
