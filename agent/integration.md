# Agent Integration Guide

This document explains how AI agents can interact with the trading system to analyze signals, generate recommendations, and control TradingView charts.

## Overview

The agent integration system allows AI agents (like Manus) to:

1. **Receive trading signals** via webhook
2. **Analyze signals** against defined strategies
3. **Generate trade recommendations** with entry/exit levels
4. **Draw on TradingView charts** automatically
5. **Monitor positions** and suggest adjustments

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Agent System                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │   Signal     │───▶│   Strategy   │───▶│   Chart      │       │
│  │   Receiver   │    │   Analyzer   │    │   Controller │       │
│  └──────────────┘    └──────────────┘    └──────────────┘       │
│         │                   │                   │                │
│         ▼                   ▼                   ▼                │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │   Signal     │    │   Trade      │    │   Drawing    │       │
│  │   Storage    │    │   Recommender│    │   Engine     │       │
│  └──────────────┘    └──────────────┘    └──────────────┘       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Agent API Reference

### Signal Analysis

#### Analyze Signal Against Strategy

```javascript
AgentAPI.analyzeSignal(signal, strategy)
```

**Parameters:**
- `signal`: Signal object from webhook
- `strategy`: Strategy JSON object

**Returns:**
```javascript
{
  matches: true,
  direction: "long",
  confluence_met: 5,
  confluence_required: 4,
  signals_present: ["blue_dot", "money_flow_green", "rsi_oversold", "bullish_divergence", "support_zone"],
  signals_missing: [],
  recommendation: {
    action: "LONG",
    confidence: 0.85,
    reasoning: "Signal meets all entry criteria with 5/4 confluence"
  }
}
```

#### Get Strategy Match Score

```javascript
AgentAPI.getMatchScore(signal, strategy)
```

**Returns:**
```javascript
{
  score: 0.85,
  max_score: 1.0,
  breakdown: {
    required_signals: 1.0,
    confluence: 0.8,
    filters: 0.75
  }
}
```

### Trade Recommendations

#### Generate Trade Recommendation

```javascript
AgentAPI.generateRecommendation(signal, strategy)
```

**Returns:**
```javascript
{
  recommendation: "LONG",
  confidence: 0.85,
  entry: {
    price: 45000,
    type: "market",
    timing: "candle_close"
  },
  stop_loss: {
    price: 44000,
    type: "swing_point",
    risk_percent: 2.2
  },
  take_profit: [
    { price: 46000, percent: 50, rr: 1.0 },
    { price: 47000, percent: 30, rr: 2.0 },
    { price: 48000, percent: 20, rr: 3.0 }
  ],
  position_size: {
    risk_amount: 200,
    units: 0.044,
    notional_value: 2000
  },
  confluence_met: [
    "blue_dot",
    "money_flow_green",
    "rsi_oversold",
    "bullish_divergence",
    "support_zone"
  ],
  reasoning: "Blue dot signal at key support ($44,800) with 5/4 confluence signals met. RSI at 28 showing oversold conditions with bullish divergence on momentum waves. Money flow transitioning to green confirms buying pressure."
}
```

### Chart Control

#### Draw Horizontal Line

```javascript
AgentAPI.chart.drawHorizontalLine(price, color, label, style)
```

**Parameters:**
- `price`: Price level for the line
- `color`: Line color (e.g., "green", "#00ff00")
- `label`: Text label for the line
- `style`: Line style ("solid", "dashed", "dotted")

#### Draw Zone

```javascript
AgentAPI.chart.drawZone(priceHigh, priceLow, color, label, opacity)
```

**Parameters:**
- `priceHigh`: Upper boundary of zone
- `priceLow`: Lower boundary of zone
- `color`: Zone color
- `label`: Text label
- `opacity`: Fill opacity (0-1)

#### Add Indicator

```javascript
AgentAPI.chart.addIndicator(name, settings)
```

**Parameters:**
- `name`: Indicator name
- `settings`: Indicator configuration object

#### Create Annotation

```javascript
AgentAPI.chart.addAnnotation(price, time, text, type)
```

**Parameters:**
- `price`: Price level
- `time`: Timestamp
- `text`: Annotation text
- `type`: Annotation type ("label", "arrow", "flag")

#### Set Symbol

```javascript
AgentAPI.chart.setSymbol(symbol, exchange)
```

#### Set Timeframe

```javascript
AgentAPI.chart.setTimeframe(timeframe)
```

### Drawing Trade Setup

#### Complete Trade Setup Drawing

```javascript
AgentAPI.chart.drawTradeSetup(recommendation)
```

This method draws the complete trade setup including:
- Entry line (green for long, red for short)
- Stop loss line (red)
- Take profit lines (blue)
- Risk/reward zones
- Labels with prices and R:R ratios

**Example:**
```javascript
const recommendation = await AgentAPI.generateRecommendation(signal, strategy);

if (recommendation.recommendation !== "NO_TRADE") {
  await AgentAPI.chart.drawTradeSetup(recommendation);
}
```

## Workflow Examples

### Example 1: Process Incoming Signal

```javascript
// 1. Receive signal from webhook
const signal = await fetch('/api/signals/latest').then(r => r.json());

// 2. Load strategy
const strategy = await fetch('/strategies/market-cipher-basic.json').then(r => r.json());

// 3. Analyze signal
const analysis = AgentAPI.analyzeSignal(signal, strategy);

// 4. If signal matches, generate recommendation
if (analysis.matches) {
  const recommendation = AgentAPI.generateRecommendation(signal, strategy);
  
  // 5. Draw on chart
  await AgentAPI.chart.setSymbol(signal.symbol, signal.exchange);
  await AgentAPI.chart.drawTradeSetup(recommendation);
  
  // 6. Return recommendation to user
  return recommendation;
}
```

### Example 2: Multi-Strategy Analysis

```javascript
// Load multiple strategies
const strategies = [
  await fetch('/strategies/market-cipher-basic.json').then(r => r.json()),
  await fetch('/strategies/algopro-quantx.json').then(r => r.json())
];

// Analyze signal against all strategies
const results = strategies.map(strategy => ({
  strategy: strategy.strategy_name,
  analysis: AgentAPI.analyzeSignal(signal, strategy),
  score: AgentAPI.getMatchScore(signal, strategy)
}));

// Find best matching strategy
const bestMatch = results.reduce((best, current) => 
  current.score.score > best.score.score ? current : best
);

// Generate recommendation from best strategy
if (bestMatch.analysis.matches) {
  return AgentAPI.generateRecommendation(signal, bestMatch.strategy);
}
```

### Example 3: Continuous Monitoring

```javascript
// Poll for new signals
setInterval(async () => {
  const signals = await fetch('/api/signals?since=' + lastCheck).then(r => r.json());
  
  for (const signal of signals.signals) {
    const analysis = AgentAPI.analyzeSignal(signal, strategy);
    
    if (analysis.matches && analysis.confluence_met >= 5) {
      // High confidence signal
      const recommendation = AgentAPI.generateRecommendation(signal, strategy);
      
      // Notify user
      await notifyUser({
        type: 'high_confidence_signal',
        signal,
        recommendation
      });
    }
  }
  
  lastCheck = Date.now();
}, 60000); // Check every minute
```

## Agent Prompts

### Analyzing a Signal

When an agent receives a signal, use this prompt structure:

```
I received a trading signal:
- Symbol: {symbol}
- Direction: {signal}
- Indicator: {indicator}
- Price: {price}
- Confluence: {confluence_count} signals

Please analyze this against my {strategy_name} strategy and provide:
1. Whether the signal meets entry criteria
2. Recommended entry, stop loss, and take profit levels
3. Position size based on 2% risk
4. Confidence level and reasoning
```

### Drawing on Chart

```
Please draw the following on my TradingView chart:
1. Entry line at {entry_price} (green)
2. Stop loss at {stop_loss} (red)
3. Take profit levels at {tp1}, {tp2}, {tp3} (blue)
4. Add labels showing R:R ratios
```

### Reviewing Trade Setup

```
Review my current trade setup:
- Entry: {entry}
- Stop Loss: {stop_loss}
- Take Profit: {take_profit}
- Current Price: {current_price}

Provide:
1. Current P&L status
2. Whether to adjust stop loss
3. Whether to take partial profits
4. Any concerns about the trade
```

## Best Practices

### 1. Validate Signals

Always validate incoming signals before processing:
- Check required fields are present
- Verify price is reasonable
- Confirm timestamp is recent

### 2. Use Confluence Thresholds

Don't trade every signal - require minimum confluence:
- Low confidence: 3-4 signals
- Medium confidence: 5-6 signals
- High confidence: 7+ signals

### 3. Respect Risk Management

Always calculate position size based on:
- Account risk percentage
- Stop loss distance
- Maximum position limits

### 4. Log Everything

Maintain logs of:
- All received signals
- Analysis results
- Recommendations generated
- Trades taken

### 5. Handle Errors Gracefully

Implement error handling for:
- Network failures
- Invalid data
- API rate limits
- Chart connection issues

## Troubleshooting

### Signal Not Processing

1. Check webhook endpoint is receiving data
2. Verify signal format matches schema
3. Confirm strategy file is valid JSON
4. Check API authentication

### Chart Not Updating

1. Verify TradingView connection
2. Check symbol/exchange format
3. Confirm chart permissions
4. Test with simple drawing first

### Recommendations Not Generating

1. Verify confluence requirements
2. Check filter conditions
3. Confirm strategy is enabled for direction
4. Review signal timestamp (not too old)
