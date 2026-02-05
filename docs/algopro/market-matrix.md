# AlgoPro Market Matrix

Market Matrix is AlgoPro's comprehensive trading dashboard that combines multiple technical indicators into a unified confluence system.

## Overview

Market Matrix is designed to help traders identify optimal entry and exit points by analyzing:
- Price action
- Volume dynamics
- Market structure
- Technical indicators

The tool adapts to your chart's visible range, providing relevant analysis whether you're zoomed in for scalping or zoomed out for swing trading.

## Core Components

### 1. Trade Confluence Assistant

The heart of Market Matrix - a dashboard that aggregates multiple signals into actionable intelligence.

#### Dashboard Layout

```
┌─────────────────────────────────────┐
│ Trade Bias & RSI Status             │
├─────────────────────────────────────┤
│ HTF Bias: LONG    │ RSI: 45 Neutral │
│ LTF Bias: LONG    │                 │
├─────────────────────────────────────┤
│ Confluence Signals                  │
├─────────────────────────────────────┤
│ Divergence:        LONG             │
│ Fair Value Gap:    NEUTRAL          │
│ VWAP Deviation:    LONG             │
│ Volume Exhaustion: LONG             │
│ Williams %R:       NEUTRAL          │
│ SFP:               LONG             │
│ Stochastic:        NEUTRAL          │
│ Liquidity Grab:    LONG             │
│ Price Exhaustion:  NEUTRAL          │
│ Fear & Greed:      LONG             │
│ Mean Reversion:    LONG             │
│ KDE RSI:           NEUTRAL          │
│ Dying Momentum:    NEUTRAL          │
├─────────────────────────────────────┤
│ Confluence: 7 LONG │ Threshold: 5   │
└─────────────────────────────────────┘
```

### 2. Confluence Signals Explained

| Signal | What It Detects | Long Trigger | Short Trigger |
|--------|-----------------|--------------|---------------|
| **Divergence** | Price vs 10 indicators | Bullish divergence | Bearish divergence |
| **Fair Value Gap** | Price imbalances | FVG below price | FVG above price |
| **VWAP Deviation** | Distance from VWAP | Extended below | Extended above |
| **Volume Exhaustion** | Volume trend ending | Selling exhaustion | Buying exhaustion |
| **Williams %R** | Momentum oscillator | Oversold (<-80) | Overbought (>-20) |
| **SFP** | Failed breakouts | Bullish SFP | Bearish SFP |
| **Stochastic** | Overbought/oversold | Oversold cross up | Overbought cross down |
| **Liquidity Grab** | Swept levels | Bullish LG | Bearish LG |
| **Price Exhaustion** | Exhaustion candles | Bullish exhaustion | Bearish exhaustion |
| **Fear & Greed** | Emotional extremes | Extreme fear | Extreme greed |
| **Mean Reversion** | Price deviation | Below mean | Above mean |
| **KDE RSI** | Enhanced RSI | Oversold | Overbought |
| **Dying Momentum** | Weakening trend | Bearish dying | Bullish dying |

### 3. Trade Bias System

**High-Timeframe (HTF) Bias:**
- Based on moving average position
- Price above MA = LONG bias
- Price below MA = SHORT bias

**Low-Timeframe (LTF) Bias:**
- Same calculation on lower timeframe
- Confirms or contradicts HTF bias

**RSI Status:**
- Above 70 = Overbought (red)
- Below 30 = Oversold (green)
- 30-70 = Neutral (gray)

### 4. Visual Alerts

Market Matrix plots circles on the chart:
- **Green circles**: Long confluence threshold met
- **Red circles**: Short confluence threshold met

Circles appear at the price level where conditions triggered.

## Additional Tools

### Volume Profile
- Shows volume distribution at price levels
- Identifies high-volume nodes (support/resistance)
- Point of Control (POC) marking

### Fibonacci Reversal Zone
- Automatic Fibonacci retracement levels
- Key reversal zones highlighted
- Works on visible chart range

### AI Support & Resistance Zones
- Machine learning-based level detection
- Adapts to current price action
- Strength rating for each zone

### Liquidity Zones
- Identifies areas of concentrated orders
- Stop loss clusters
- Potential reversal points

### Market Structure & Fractals
- Swing high/low identification
- Break of structure signals
- Change of character detection

### Divergence Detector
- Scans 10 different indicators
- Automatic divergence marking
- Bullish/bearish classification

### Swing Failure Patterns (SFP)
- Failed breakout detection
- High-probability reversal signals
- Works on all timeframes

### Liquidity Grab (LG)
- Identifies stop hunts
- Swept level detection
- Reversal confirmation

### VWAP
- Volume Weighted Average Price
- Standard deviation bands
- Intraday anchor points

## Configuration Guide

### Confluence Settings

```
Recommended Settings:
├── Long Threshold: 5-6 signals
├── Short Threshold: 5-6 signals
├── Show Dashboard: Yes
├── Dashboard Position: Top Right
└── Alert on Confluence: Yes
```

### Visual Settings

```
Display Options:
├── Show Volume Profile: Yes
├── Show S/R Zones: Yes
├── Show Liquidity Zones: Yes
├── Show Divergences: Yes
├── Show SFP: Yes
└── Show LG: Yes
```

## Trading with Market Matrix

### Step-by-Step Process

1. **Check HTF/LTF Bias**
   - Both LONG = Look for longs only
   - Both SHORT = Look for shorts only
   - Mixed = Wait for alignment or trade with caution

2. **Monitor Confluence Count**
   - Wait for threshold to be met
   - Higher count = higher probability

3. **Confirm with Visual Tools**
   - Price at S/R zone?
   - Volume profile showing interest?
   - Liquidity grab occurred?

4. **Enter on Confirmation**
   - Confluence circle appears
   - Price action confirms direction
   - Risk defined by nearby structure

5. **Manage Trade**
   - Watch for opposite confluence
   - Use S/R zones for targets
   - Trail stop using structure

### Example Trade Setup

```
Scenario: Long Trade on ES Futures

1. HTF Bias: LONG (price above 4H MA)
2. LTF Bias: LONG (price above 15M MA)
3. RSI: 35 (approaching oversold)

Confluence Signals (7/13):
✓ Divergence: Bullish
✓ VWAP Deviation: Extended below
✓ Volume Exhaustion: Selling exhaustion
✓ SFP: Bullish swing failure
✓ Liquidity Grab: Bullish LG
✓ Fear & Greed: Extreme fear
✓ Mean Reversion: Below mean

Additional Confirmation:
✓ Price at AI S/R zone
✓ Volume profile POC nearby
✓ Green confluence circle appeared

Entry: 4520
Stop Loss: 4510 (below LG low)
Take Profit 1: 4540 (2:1 R:R)
Take Profit 2: 4560 (4:1 R:R)
```

## Combining with Other AlgoPro Tools

### Market Matrix + QuantX

```
Workflow:
1. Use Matrix for visual analysis
2. Wait for QuantX signal
3. Confirm Matrix confluence meets threshold
4. Enter with QuantX risk management
```

### Market Matrix + V1.5

```
Workflow:
1. V1.5 identifies optimized entry
2. Matrix confirms confluence
3. Enter on V1.5 signal
4. Use Matrix S/R for targets
```

## Alert Configuration

### Confluence Alert

```json
{
  "indicator": "algopro_market_matrix",
  "alert_type": "confluence",
  "symbol": "{{ticker}}",
  "direction": "long|short",
  "confluence_count": 7,
  "signals": [
    "divergence",
    "vwap_deviation",
    "volume_exhaustion",
    "sfp",
    "liquidity_grab",
    "fear_greed",
    "mean_reversion"
  ],
  "price": {{close}},
  "timestamp": "{{timenow}}"
}
```

## Best Practices

1. **Don't trade every confluence signal** - Wait for price action confirmation
2. **Higher confluence = better trades** - Be patient for 6+ signals
3. **Respect the bias** - Don't fight HTF/LTF alignment
4. **Use S/R zones** - Confluence at key levels is strongest
5. **Combine with price action** - Candlestick patterns add confirmation
6. **Manage risk always** - No signal is 100% accurate
