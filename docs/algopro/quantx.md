# AlgoPro QuantX

QuantX is AlgoPro's flagship tool for futures trading, designed specifically for NQ (Nasdaq) and ES (S&P 500) futures markets.

## Overview

QuantX revolutionizes trading by leveraging a confluence of signals for precision entries. Unlike traditional trend-following systems, QuantX waits for multiple favorable conditions to align before triggering trades.

## Core Concept: Buy Low, Sell High

QuantX is designed to enter at optimal points:
- **Buying low** in uptrends (at pullbacks)
- **Selling high** in downtrends (at rallies)

This approach creates a larger "profit cushion" compared to chasing price or entering mid-move.

## Key Features

### 1. Confluence Signal System

QuantX requires multiple conditions to align before entry:

```
Confluence Signals Include:
├── Divergence detection
├── Fair Value Gaps (FVG)
├── VWAP deviation
├── Volume exhaustion
├── Williams %R
├── Swing Failure Patterns
├── Stochastic signals
├── Liquidity grabs
├── Price exhaustion
├── Fear & Greed zones
├── Mean reversion
├── KDE RSI
└── Dying momentum
```

### 2. Configurable Entry Requirements

| Setting | Description | Range |
|---------|-------------|-------|
| Long Confluence | Signals needed for long | 1-13 |
| Short Confluence | Signals needed for short | 1-13 |
| Signal Window | Bars to wait for alignment | 1-50 |
| Breakout Required | Wait for trend breakout | Yes/No |
| Breakout Sensitivity | How strong breakout must be | 1-10 |
| Cooldown Period | Bars before next trade | 0-100 |

### 3. Risk Management System

**Stop Loss Options:**

| Type | Description | Best For |
|------|-------------|----------|
| Fixed Dollar | Exact dollar amount | Consistent risk |
| Fixed Percentage | % of entry price | Proportional risk |
| Dynamic ATR | Volatility-based | Adaptive risk |

**ATR Stop Loss Features:**
- Adapts to current volatility
- Percentage cap prevents oversized stops
- Example: 1% cap means stop never exceeds 1% of entry

**Take Profit:**
- Based on Risk/Reward ratio
- Example: $5 stop with 2:1 R:R = $10 take profit

### 4. Session & Time Filters

Control when trades are allowed:

```
Session Management:
├── Close trades at session end
├── Minute-level session control (5-min increments)
├── Time filter start/end
└── Restrict trading to specific hours
```

### 5. Backtesting Dashboard

Performance metrics displayed:

| Metric | Description |
|--------|-------------|
| Initial/Current Equity | Account growth |
| Multiplier | Equity growth factor |
| Total Trades | Number of trades taken |
| Win Rate | Percentage of winning trades |
| Avg Win/Loss | Average trade performance |
| Max Drawdown | Largest equity decline |
| Monthly/Annual Returns | Periodic performance |
| Risk/Reward Ratio | Actual R:R achieved |
| Required Win Rate | Break-even win rate |
| Win Rate Differential | Current vs required |
| Strategy Grade | A-F performance rating |

## Configuration Guide

### Recommended Settings for NQ

```
Confluence Settings:
├── Long Confluence: 4-5
├── Short Confluence: 4-5
├── Signal Window: 10 bars
├── Breakout Required: Yes
└── Breakout Sensitivity: 5

Risk Settings:
├── Stop Loss Type: ATR
├── ATR Cap: 0.5%
└── Risk/Reward: 2.0

Session Settings:
├── Close at Session End: Yes
├── Time Filter: 9:30 AM - 4:00 PM ET
└── Cooldown: 5 bars
```

### Recommended Settings for ES

```
Confluence Settings:
├── Long Confluence: 5-6
├── Short Confluence: 5-6
├── Signal Window: 15 bars
├── Breakout Required: Yes
└── Breakout Sensitivity: 4

Risk Settings:
├── Stop Loss Type: ATR
├── ATR Cap: 0.4%
└── Risk/Reward: 2.5

Session Settings:
├── Close at Session End: Yes
├── Time Filter: 9:30 AM - 4:00 PM ET
└── Cooldown: 10 bars
```

## Trading with QuantX

### Entry Process

1. **Wait for Confluence**
   - Dashboard shows current confluence count
   - Green = Long conditions met
   - Red = Short conditions met

2. **Check Breakout (if enabled)**
   - Wait for price to break trend line
   - Sensitivity determines required strength

3. **Enter on Signal**
   - Entry triggered when all conditions met
   - Stop loss and take profit automatically calculated

4. **Manage Trade**
   - Monitor dashboard for exit signals
   - Let take profit or stop loss execute

### Exit Process

1. **Take Profit Hit** - Trade closes at target
2. **Stop Loss Hit** - Trade closes at risk level
3. **Session End** - Trade closes if enabled
4. **Opposite Signal** - Optional exit on reversal

## Combining with Market Matrix

QuantX pairs perfectly with Market Matrix for enhanced confluence:

```
QuantX + Market Matrix Setup:
├── Use Market Matrix for visual confluence
├── QuantX for automated entries
├── Market Matrix confirms QuantX signals
└── Higher confluence = higher probability
```

### Integration Benefits

1. **Visual Confirmation** - See all confluence factors
2. **Manual Override** - Skip signals that don't look right
3. **Additional Filters** - Use Matrix levels for entries
4. **Better Exits** - Matrix shows exhaustion zones

## Troubleshooting

### Runtime Error

If you see a runtime error:
- Stop loss may be too large for the asset
- Reduce stop loss size
- Disable "Show Take Profit/Stop Loss" to debug

### No Signals Appearing

- Confluence requirement may be too high
- Reduce required signals
- Check if time filter is blocking signals
- Verify breakout settings aren't too strict

### Too Many Signals

- Increase confluence requirement
- Enable breakout filter
- Increase cooldown period
- Add time filter restrictions

## Alert Setup for Automation

```json
{
  "indicator": "algopro_quantx",
  "signal": "{{strategy.order.action}}",
  "symbol": "{{ticker}}",
  "price": {{close}},
  "confluence": {
    "long_count": "{{plot_0}}",
    "short_count": "{{plot_1}}"
  },
  "risk": {
    "stop_loss": "{{plot_2}}",
    "take_profit": "{{plot_3}}"
  }
}
```
