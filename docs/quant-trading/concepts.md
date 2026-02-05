# Quantitative Trading Concepts

> **Source**: Compiled from Roman Paolucci, DeltaTrend Trading, neurotrader, and Moon Dev educational content

## Core Principles

### Trading vs Gambling

| Aspect | Gambling | Trading |
|--------|----------|---------|
| **Expected Value** | Negative (house edge) | Can be positive (with skill) |
| **Skill Factor** | Minimal | Significant |
| **Long-term Outcome** | Guaranteed loss | Potential profit |

**Key Insight**: Trading becomes gambling when you don't have a proven edge. The difference is whether your expected value is positive or negative.

---

## Ergodicity and Non-Ergodic Systems

### What is Ergodicity?
A system is **ergodic** when the time average equals the ensemble average. Most trading scenarios are **non-ergodic**.

### Time Average vs Ensemble Average

| Type | Description | Example |
|------|-------------|---------|
| **Ensemble Average** | Average across many traders at one time | "50% of traders made money today" |
| **Time Average** | Average for one trader over time | "Over 10 years, this trader's CAGR was X%" |

### Why This Matters
- Survivorship bias makes trading look more profitable than it is
- Your individual wealth path matters more than average outcomes
- Ruin (going to zero) is permanent - you can't recover

---

## Position Sizing: The Kelly Criterion

### The Formula
```
f = (p - q) / b

Where:
f = fraction of bankroll to bet
p = probability of winning
q = probability of losing (1 - p)
b = odds received on the bet (win amount / loss amount)
```

### Example
```
Win rate: 55% (p = 0.55)
Loss rate: 45% (q = 0.45)
Risk/Reward: 2:1 (b = 2)

f = (0.55 - 0.45) / 2 = 0.05 = 5%
```

### Half Kelly (Recommended)
- Full Kelly is mathematically optimal but volatile
- **Half Kelly** provides smoother wealth path
- Reduces risk of ruin significantly
- Sacrifice some returns for stability

### Key Rules
1. **Never bet fixed amounts** - bet percentages
2. **Have an edge first** - Kelly assumes positive EV
3. **Be conservative** - use Half Kelly or less
4. **Avoid excessive leverage** - amplifies mistakes

---

## Strategy Development Process

### The Quant Process (DeltaTrend)

1. **Ideation & Feature Engineering**
   - Generate trading ideas
   - Define entry/exit rules
   - Select indicators and parameters

2. **Programmatic Backtesting**
   - Test strategy on historical data
   - Calculate performance metrics
   - Identify potential issues

3. **Statistical Validation**
   - Monte Carlo simulation
   - Permutation tests
   - Walk-forward analysis

4. **Deployment**
   - Paper trading first
   - Small position sizes
   - Scale up gradually

### Avoiding Data Mining Bias

| Problem | Solution |
|---------|----------|
| **Overfitting** | Out-of-sample testing |
| **Curve fitting** | Walk-forward validation |
| **Cherry picking** | Monte Carlo permutation |
| **In-sample excellence** | Multiple validation methods |

---

## Statistical Validation Methods

### Monte Carlo Permutation Test
1. Shuffle the order of trades randomly
2. Calculate performance on shuffled data
3. Repeat thousands of times
4. Compare original performance to distribution
5. Calculate p-value

**Interpretation**: If your strategy beats 95% of random shuffles, it likely has real edge (p < 0.05).

### Walk-Forward Analysis
1. Optimize on training period
2. Test on out-of-sample period
3. Roll forward and repeat
4. Aggregate out-of-sample results

**Purpose**: Simulates real trading where you optimize on past data and trade into the unknown future.

### Walk-Forward Monte Carlo
Combines both methods:
1. Perform walk-forward analysis
2. Apply Monte Carlo permutation to each walk-forward period
3. Most rigorous validation approach

---

## Market Regime Analysis

### Markov Chains for Trading

A Markov Chain models market states and transition probabilities.

#### Components
| Element | Description |
|---------|-------------|
| **States** | Market conditions (bull, bear, sideways) |
| **Transition Matrix** | Probabilities of moving between states |
| **State Vector** | Current probability distribution |
| **Steady-State** | Long-term equilibrium probabilities |

#### Application
1. Define market regimes (e.g., based on volatility, trend)
2. Calculate transition probabilities from historical data
3. Find steady-state distribution
4. Set base exposure based on steady-state probabilities

### Example Regime Definition
```
State 1: Low volatility, uptrend
State 2: Low volatility, downtrend
State 3: High volatility, uptrend
State 4: High volatility, downtrend
```

---

## Options Trading: Black-Scholes Model

### Key Inputs
| Input | Description |
|-------|-------------|
| **S** | Underlying asset price |
| **K** | Strike price |
| **σ** | Volatility |
| **r** | Interest rate |
| **T** | Time until maturity |

### Trading Edge from Mispricing
1. Calculate theoretical option value using Black-Scholes
2. Compare to market price
3. If theoretical > market: option is undervalued (buy)
4. If theoretical < market: option is overvalued (sell)

### Statistical Arbitrage
- Execute many trades based on mispricing
- Individual trades may lose
- Law of large numbers: positive EV materializes over time
- **Key**: Survive short-term drawdowns to reach long-term profitability

---

## Risk Management Principles

### Core Rules

1. **Survive First**
   - Capital preservation is priority #1
   - You can't trade if you're wiped out
   - Ruin is permanent

2. **Size Appropriately**
   - Use Kelly Criterion or more conservative
   - Never risk more than you can afford to lose
   - Scale position size to conviction

3. **Diversify**
   - Don't put all capital in one trade
   - Spread across uncorrelated strategies
   - Manage portfolio-level risk

4. **Cut Losses**
   - Have predefined stop losses
   - Don't let losers run
   - Accept small losses to avoid large ones

### Risk Metrics to Track

| Metric | Description | Target |
|--------|-------------|--------|
| **Max Drawdown** | Largest peak-to-trough decline | < 20% |
| **Sharpe Ratio** | Risk-adjusted returns | > 1.0 |
| **Win Rate** | Percentage of winning trades | Context-dependent |
| **Risk/Reward** | Average win / average loss | > 2:1 |
| **Profit Factor** | Gross profit / gross loss | > 1.5 |

---

## Quant App Integration (Moon Dev)

### Features
- Risk management enforcement
- Trade execution automation
- Webhook integration with TradingView
- Position sizing controls

### Configuration
```json
{
  "max_leverage": 5,
  "max_position_percent": 10,
  "max_loss_per_trade_percent": 1,
  "daily_loss_limit_percent": 5
}
```

### Webhook Format
```json
{
  "symbol": "BTCUSDT",
  "side": "long",
  "type": "market",
  "quantity": 0.1,
  "price": 97000
}
```

---

## Key Takeaways

1. **Have an Edge**: Don't trade without a statistically validated strategy
2. **Size Correctly**: Use Kelly Criterion or more conservative
3. **Validate Rigorously**: Monte Carlo, walk-forward, multiple tests
4. **Manage Risk**: Survival is more important than returns
5. **Think Long-term**: Focus on expected value over many trades
6. **Avoid Gurus**: Win rate alone means nothing without context
7. **Use Data**: Let statistics guide decisions, not emotions
