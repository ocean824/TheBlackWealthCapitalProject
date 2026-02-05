# Signal Strength & Confluence Scoring System

> **Purpose**: A unified scoring framework that weights all signals across Market Cipher, AlgoPro, quant factors, and risk metrics to determine trade execution decisions.

---

## Table of Contents

1. [Overview](#overview)
2. [Market Cipher Signal Scores](#market-cipher-signal-scores)
3. [AlgoPro Signal Scores](#algopro-signal-scores)
4. [Quant Factor Scores](#quant-factor-scores)
5. [Risk & Liquidation Modifiers](#risk--liquidation-modifiers)
6. [Confluence Thresholds](#confluence-thresholds)
7. [Scoring Examples](#scoring-examples)
8. [Implementation JSON Schema](#implementation-json-schema)

---

## Overview

### How It Works

Every signal detected is assigned a **base score** based on its historical reliability and strength. Scores are then modified by:
- **Timeframe multiplier** (higher TF = stronger signal)
- **Confluence bonus** (multiple signals aligning)
- **Risk penalty** (liquidation zones, high leverage)
- **Quant validation** (statistical confidence)

### Score Ranges

| Total Score | Decision | Action |
|-------------|----------|--------|
| **100+** | STRONG ENTRY | Execute with full position size |
| **75-99** | GOOD ENTRY | Execute with 75% position size |
| **50-74** | MODERATE | Execute with 50% position size, tighter stops |
| **25-49** | WEAK | Paper trade only, or very small size |
| **0-24** | NO TRADE | Do not enter, wait for better setup |
| **Negative** | AVOID | Conflicting signals, stay out |

---

## Market Cipher Signal Scores

### Primary Signals (Market Cipher B)

| Signal | Base Score | Direction | Notes |
|--------|------------|-----------|-------|
| **Green Dot** | +35 | LONG | THE most important bullish signal |
| **Red Dot** | +35 | SHORT | Bearish equivalent of green dot |
| **Deep Anchor Wave** | +20 | Both | "Fat and deep" wave establishing momentum |
| **Trigger Wave Confirmation** | +15 | Both | Light→dark blue transition |
| **Higher Low on Waves (with gap)** | +15 | LONG | Key bullish structure |
| **Lower High on Waves (with gap)** | +15 | SHORT | Key bearish structure |
| **Money Flow Smooth & Directional** | +10 | Both | Green = long, Red = short |
| **Money Flow Choppy** | -25 | Both | AVOID trading |
| **RSI Green & Tight** | +10 | LONG | Building bullish strength |
| **RSI Red & Spreading** | +10 | SHORT | Bearish momentum |
| **VWAP Below Blue Waves** | +8 | LONG | Supports long entries |
| **VWAP Above Blue Waves** | +8 | SHORT | Supports short entries |

### Warning Signals (Market Cipher A)

| Signal | Base Score | Direction | Notes |
|--------|------------|-----------|-------|
| **Blood Diamond** | -50 | EXIT LONG | THE harshest bearish warning - EXIT IMMEDIATELY |
| **Yellow X** | -35 | EXIT/CAUTION | Whale manipulation warning |
| **Red X** | -25 | EXIT/SHORT | Hard warning of downward pressure |
| **Small Red Diamond** | -15 | CAUTION | Early warning signal |
| **Blue Triangle** | +/-10 | REVERSAL | Potential trend reversal point |
| **Yellow Diamond** | +5 | CONTINUATION | Trend continuation signal |

### DBSI Scoring

| DBSI Value | Score Modifier | Notes |
|------------|----------------|-------|
| **27+** | +30 | Extremely strong signal |
| **20-26** | +25 | Very strong signal |
| **14-19** | +20 | Strong signal (entry threshold) |
| **8-13** | +10 | Moderate signal |
| **1-7** | +5 | Weak signal |
| **0** | 0 | Neutral |
| **-1 to -5** | -5 | Weak opposing (good for entry if other side strong) |
| **Below -5** | -15 | Strong opposing pressure |

### 200 EMA Position (from DBSI)

| Condition | Score Modifier | Notes |
|-----------|----------------|-------|
| **Price Above 200 EMA (Green)** | +15 for LONG | Only take longs |
| **Price Below 200 EMA (Red)** | +15 for SHORT | Only take shorts |
| **Wrong side of 200 EMA** | -30 | Trading against major trend |

---

## AlgoPro Signal Scores

### QuantX Signals

| Signal | Base Score | Timeframe Multiplier | Notes |
|--------|------------|---------------------|-------|
| **QuantX Long Entry** | +25 | See below | Confluence-based futures signal |
| **QuantX Short Entry** | +25 | See below | Confluence-based futures signal |
| **QuantX Exit Signal** | -20 | N/A | Close position |
| **Breakout Confirmed** | +15 | See below | Optional breakout filter passed |
| **Confluence Count 5+** | +20 | N/A | Maximum confluence achieved |
| **Confluence Count 3-4** | +10 | N/A | Good confluence |
| **Confluence Count 1-2** | +5 | N/A | Minimum confluence |

### QuantFlow Signals

| Signal | Base Score | Notes |
|--------|------------|-------|
| **SMC Order Block Entry** | +20 | Smart Money Concepts |
| **ATR Trailing Active** | +10 | Proper risk management |
| **Institutional Flow Detected** | +15 | Big money moving |

### V1.5 Signals

| Signal | Base Score | Notes |
|--------|------------|-------|
| **V1.5 Long Signal** | +20 | Self-optimizing entry |
| **Volume Trend Support** | +10 | Dynamic support cloud |
| **Retest Signal** | +15 | Price retesting support |

### Market Matrix Signals

| Signal | Base Score | Notes |
|--------|------------|-------|
| **618 Fibonacci Level** | +15 | Key retracement level |
| **Liquidity Zone Identified** | +10 | Potential reversal area |
| **POC (Point of Control)** | +12 | Volume profile center |
| **Fair Value Gap** | +10 | Price imbalance |
| **Swing Failure Pattern** | +18 | Failed breakout reversal |
| **Liquidity Grab** | +15 | Swept level reversal |

### Timeframe Multipliers (AlgoPro)

| Timeframe | Multiplier | Notes |
|-----------|------------|-------|
| **Monthly** | 2.0x | Strongest signals |
| **Weekly** | 1.75x | Very strong |
| **Daily** | 1.5x | Strong |
| **4H** | 1.25x | Standard |
| **1H** | 1.0x | Base |
| **30M** | 0.75x | Reduced weight |
| **15M** | 0.5x | Scalp only |
| **5M** | 0.25x | Requires HTF confirmation |

---

## Quant Factor Scores

### Statistical Validation

| Factor | Score Modifier | Threshold |
|--------|----------------|-----------|
| **Monte Carlo Top 5%** | +25 | Strategy in top 5% of simulations |
| **Monte Carlo Top 10%** | +15 | Strategy in top 10% |
| **Monte Carlo Top 25%** | +5 | Strategy in top 25% |
| **Walk-Forward Passed** | +20 | Out-of-sample validation |
| **P-Value < 0.01** | +15 | Highly significant |
| **P-Value < 0.05** | +10 | Statistically significant |
| **P-Value > 0.05** | -10 | Not significant |

### Kelly Criterion Position Sizing

| Kelly Recommendation | Score Modifier | Notes |
|---------------------|----------------|-------|
| **Full Kelly Suggests Entry** | +15 | Positive expected value |
| **Half-Kelly Suggests Entry** | +10 | Conservative positive EV |
| **Kelly Suggests No Bet** | -20 | Negative expected value |

### Market Regime (Markov Chain)

| Regime | Score Modifier | Notes |
|--------|----------------|-------|
| **Bull Regime + Long Signal** | +15 | Aligned with regime |
| **Bear Regime + Short Signal** | +15 | Aligned with regime |
| **Sideways Regime** | -10 | Reduced position size |
| **Against Regime** | -25 | Trading against market state |

### Win Rate & Expected Value

| Metric | Score Modifier | Notes |
|--------|----------------|-------|
| **Win Rate > 60%** | +10 | High probability |
| **Win Rate 50-60%** | +5 | Acceptable |
| **Win Rate < 50%** | -10 | Needs high R:R |
| **Positive EV Confirmed** | +15 | Expected value > 0 |
| **Sharpe Ratio > 2** | +10 | Excellent risk-adjusted |
| **Sharpe Ratio 1-2** | +5 | Good risk-adjusted |

---

## Risk & Liquidation Modifiers

### Liquidation Zone Proximity

| Condition | Score Modifier | Notes |
|-----------|----------------|-------|
| **Entry near liquidation zone** | -20 | High risk of stop hunt |
| **Stop loss in liquidation zone** | -30 | Very likely to get stopped |
| **Clear of liquidation zones** | +10 | Safer entry |
| **Yellow X near entry** | -25 | Whale manipulation likely |

### Leverage Risk

| Leverage | Score Modifier | Notes |
|----------|----------------|-------|
| **1-3x** | +10 | Conservative, safe |
| **4-5x** | +5 | Moderate risk |
| **6-10x** | 0 | Standard futures |
| **11-20x** | -10 | High risk |
| **21-50x** | -25 | Very high risk |
| **50x+** | -40 | Extreme risk, not recommended |

### Risk/Reward Ratio

| R:R Ratio | Score Modifier | Notes |
|-----------|----------------|-------|
| **4:1+** | +15 | Excellent |
| **3:1** | +10 | Very good |
| **2:1** | +5 | Standard minimum |
| **1.5:1** | 0 | Acceptable |
| **1:1** | -10 | Poor, avoid |
| **< 1:1** | -25 | Do not take |

### Position in Trend

| Position | Score Modifier | Notes |
|----------|----------------|-------|
| **Early momentum (with confirmation)** | +10 | Best entries |
| **Mid momentum** | +5 | Good entries |
| **Late momentum** | -15 | Take profits, don't enter |
| **Counter-trend** | -20 | High risk |

---

## Confluence Thresholds

### Entry Decision Matrix

| Total Score | Position Size | Stop Loss | Take Profit | Notes |
|-------------|---------------|-----------|-------------|-------|
| **100+** | 100% | Standard | 3:1 minimum | High confidence entry |
| **85-99** | 75% | Standard | 2.5:1 minimum | Good entry |
| **70-84** | 50% | Tighter | 2:1 minimum | Moderate entry |
| **55-69** | 25% | Very tight | 2:1 minimum | Small position only |
| **40-54** | Paper trade | N/A | N/A | Practice only |
| **< 40** | NO TRADE | N/A | N/A | Wait for better setup |

### Minimum Requirements for Entry

**LONG Entry Requirements:**
- [ ] Total Score ≥ 55
- [ ] Price above 200 EMA (or score penalty accepted)
- [ ] No Blood Diamond present
- [ ] No Yellow X in last 3 candles
- [ ] Money flow NOT choppy
- [ ] At least one primary signal (Green Dot, Deep Anchor, or DBSI 14+)

**SHORT Entry Requirements:**
- [ ] Total Score ≥ 55
- [ ] Price below 200 EMA (or score penalty accepted)
- [ ] At least one bearish warning OR Red Dot
- [ ] Money flow red or turning red
- [ ] At least one primary signal (Red Dot, DBSI 14+, or Blood Diamond exit)

---

## Scoring Examples

### Example 1: Strong Long Setup (Score: 108)

| Signal | Score | Notes |
|--------|-------|-------|
| Green Dot | +35 | Primary bullish signal |
| Deep Anchor Wave | +20 | Fat and deep wave |
| Trigger Wave Confirmation | +15 | Light→dark blue |
| Money Flow Smooth Green | +10 | Directional |
| RSI Green & Tight | +10 | Building strength |
| DBSI at 18 | +20 | Strong (14+) |
| Price Above 200 EMA | +15 | Bullish trend |
| 4H Timeframe | x1.25 | Multiplier applied to AlgoPro |
| Clear of Liquidation Zones | +10 | Safe entry |
| R:R 3:1 | +10 | Very good |
| Monte Carlo Top 10% | +15 | Statistically validated |
| **TOTAL** | **108** | **STRONG ENTRY - Full size** |

### Example 2: Weak Setup with Warnings (Score: 12)

| Signal | Score | Notes |
|--------|-------|-------|
| Green Dot | +35 | Primary bullish signal |
| Yellow X Present | -35 | Whale manipulation warning |
| Money Flow Choppy | -25 | AVOID signal |
| DBSI at 8 | +10 | Moderate only |
| Price Above 200 EMA | +15 | Bullish trend |
| Near Liquidation Zone | -20 | High risk |
| 15M Timeframe | x0.5 | Low timeframe penalty |
| **TOTAL** | **12** | **NO TRADE - Too many warnings** |

### Example 3: Blood Diamond Exit (Score: -45)

| Signal | Score | Notes |
|--------|-------|-------|
| Blood Diamond | -50 | HARSHEST WARNING |
| Yellow X | -35 | Manipulation |
| DBSI Weakening (27→8) | -10 | Losing strength |
| Late Momentum Phase | -15 | Already extended |
| **TOTAL** | **-45** | **EXIT IMMEDIATELY** |

---

## Implementation JSON Schema

```json
{
  "signal_evaluation": {
    "timestamp": "2024-01-15T10:30:00Z",
    "symbol": "BTCUSDT",
    "timeframe": "4H",
    
    "market_cipher_signals": {
      "green_dot": { "present": true, "score": 35 },
      "blood_diamond": { "present": false, "score": 0 },
      "yellow_x": { "present": false, "score": 0 },
      "deep_anchor_wave": { "present": true, "score": 20 },
      "trigger_wave": { "present": true, "score": 15 },
      "money_flow": { "state": "smooth_green", "score": 10 },
      "rsi": { "state": "green_tight", "score": 10 },
      "dbsi_value": 18,
      "dbsi_score": 20,
      "ema_200_position": "above",
      "ema_200_score": 15
    },
    
    "algopro_signals": {
      "quantx_entry": { "present": true, "base_score": 25 },
      "confluence_count": 4,
      "confluence_score": 10,
      "timeframe_multiplier": 1.25
    },
    
    "quant_factors": {
      "monte_carlo_percentile": 8,
      "monte_carlo_score": 15,
      "kelly_recommendation": "half_kelly",
      "kelly_score": 10,
      "market_regime": "bull",
      "regime_alignment_score": 15
    },
    
    "risk_factors": {
      "liquidation_zone_proximity": "clear",
      "liquidation_score": 10,
      "leverage": 5,
      "leverage_score": 5,
      "risk_reward_ratio": 3.0,
      "rr_score": 10,
      "momentum_phase": "mid",
      "momentum_score": 5
    },
    
    "total_score": 108,
    "decision": "STRONG_ENTRY",
    "position_size_percent": 100,
    "recommended_action": "ENTER_LONG",
    
    "entry_checklist": {
      "score_above_55": true,
      "above_200_ema": true,
      "no_blood_diamond": true,
      "no_recent_yellow_x": true,
      "money_flow_not_choppy": true,
      "primary_signal_present": true,
      "all_requirements_met": true
    }
  }
}
```

---

## Quick Reference Card

### Strongest Bullish Confluence (Target: 100+)
- Green Dot (+35)
- Deep Anchor + Trigger Wave (+35)
- DBSI 14+ (+20)
- Above 200 EMA (+15)
- Smooth Money Flow (+10)
- Clear of Liquidation (+10)

### Strongest Bearish Confluence (Target: 100+)
- Blood Diamond (+50 for SHORT)
- Yellow X (+35 for SHORT)
- DBSI 14+ Bearish (+20)
- Below 200 EMA (+15)
- Red Money Flow (+10)
- Liquidation Sweep Complete (+10)

### Automatic NO TRADE Conditions
- Blood Diamond present (for longs)
- Money Flow Choppy
- Score below 40
- Against 200 EMA without acceptance
- In liquidation zone
- Yellow X in last 3 candles

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024-01 | Initial scoring system |
| 1.1 | 2024-02 | Added AlgoPro timeframe multipliers |
| 1.2 | 2024-02 | Added quant validation factors |
| 1.3 | 2024-02 | Added liquidation zone modifiers |
