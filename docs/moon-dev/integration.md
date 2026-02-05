# Moon Dev Integration Guide

> **Source**: Analyzed from moon-dev-ai-agents and moon-dev-trading-bots repositories

## Overview

Moon Dev provides a suite of AI-powered trading agents and bots that can be integrated with this system for automated analysis, risk management, and trade execution.

---

## Moon Dev AI Agents

### Available Agents

| Agent | Purpose | Key Features |
|-------|---------|--------------|
| **Trading Agent** | Trade execution | Multi-exchange support, position management |
| **Chart Analysis Agent** | Technical analysis | AI-powered chart reading, pattern recognition |
| **Risk Agent** | Risk management | Portfolio monitoring, limit enforcement |
| **Swarm Agent** | Multi-model consensus | Queries multiple AI models for decisions |
| **Copy Trade Agent** | Social trading | Follows successful traders |
| **Sentiment Agent** | Market sentiment | Social media and news analysis |

---

## Trading Agent

### Core Functionality
- Executes trades across multiple exchanges
- Manages open positions
- Handles order lifecycle

### Integration Points
```python
# Example: Trading Agent webhook receiver
{
  "action": "trade",
  "symbol": "BTCUSDT",
  "side": "long",
  "type": "market",
  "quantity": 0.1,
  "stop_loss": 95000,
  "take_profit": 105000
}
```

---

## Chart Analysis Agent

### AI-Powered Analysis
- Uses vision models to analyze chart images
- Identifies patterns and setups
- Provides trade recommendations

### Analysis Output
```json
{
  "analysis": {
    "trend": "bullish",
    "patterns": ["higher_lows", "ascending_triangle"],
    "support_levels": [95000, 93500],
    "resistance_levels": [100000, 102000],
    "recommendation": "LONG",
    "confidence": 0.78,
    "entry_zone": [96000, 97000],
    "stop_loss": 94500,
    "take_profit": [100000, 102000]
  }
}
```

### Integration with TradingView
1. Capture chart screenshot via TradingView API
2. Send to Chart Analysis Agent
3. Receive structured analysis
4. Display on dashboard with annotations

---

## Risk Agent

### Risk Management Features

#### Portfolio Monitoring
- Tracks total portfolio value
- Monitors individual position sizes
- Calculates real-time P&L

#### Limit Enforcement
| Limit Type | Description |
|------------|-------------|
| **Max Loss** | Daily loss limit (% of portfolio) |
| **Max Gain** | Daily profit target |
| **Position Size** | Max % per position |
| **Leverage** | Maximum allowed leverage |

#### Override System
The Risk Agent can analyze market conditions to decide whether to override limits:

**Max Loss Override** (Conservative):
- Requires 90%+ confidence
- All positions must show reversal potential
- Strong reversal signals required

**Max Gain Override** (More Lenient):
- Requires 60%+ confidence
- Most positions should show upward momentum
- Continued momentum signals

### Risk Override Prompt
```
Analyze the provided market data for each position and decide if we should override the daily limit.

Consider for each position:
1. Recent price action and momentum (both 15m and 5m timeframes)
2. Volume patterns and trends
3. Market conditions and volatility
4. Risk/reward ratio based on current position size
```

---

## Swarm Agent

### Multi-Model Consensus

The Swarm Agent queries multiple AI models in parallel and synthesizes responses for trading decisions.

#### Supported Models
| Provider | Model | Use Case |
|----------|-------|----------|
| DeepSeek | deepseek-chat | Fast analysis |
| xAI | grok-4-fast-reasoning | Quick reasoning |
| OpenRouter | qwen/qwen3-max | Powerful reasoning |
| Claude | claude-sonnet-4-5 | Latest capabilities |
| Claude | claude-opus-4-5 | Most powerful |
| OpenRouter | openai/gpt-5-mini | GPT-5 via OpenRouter |
| OpenRouter | z-ai/glm-4.6 | Zhipu AI reasoning |

#### Consensus Generation
1. Send prompt to all enabled models
2. Collect individual responses
3. Synthesize consensus summary (3 sentences max)
4. Highlight agreements and disagreements

#### Usage Example
```python
from src.agents.swarm_agent import SwarmAgent

swarm = SwarmAgent()
result = swarm.query("Should I buy Bitcoin at $97,000?")

# Access consensus
print(result["consensus_summary"])

# Access individual responses
for provider, data in result["responses"].items():
    if data["success"]:
        print(f"{provider}: {data['response']}")
```

---

## Integration Architecture

### Signal Flow
```
TradingView Alert
       ↓
   Webhook Endpoint
       ↓
   Signal Processing
       ↓
   ┌───────────────────────────────────────┐
   │           Agent Analysis              │
   │  ┌─────────┐ ┌─────────┐ ┌─────────┐ │
   │  │ Chart   │ │  Risk   │ │  Swarm  │ │
   │  │ Agent   │ │  Agent  │ │  Agent  │ │
   │  └────┬────┘ └────┬────┘ └────┬────┘ │
   │       └───────────┼───────────┘      │
   │                   ↓                   │
   │         Consensus Decision            │
   └───────────────────────────────────────┘
       ↓
   Trade Execution (if approved)
       ↓
   Position Monitoring
```

### Webhook Receiver Enhancement
```typescript
// Enhanced webhook with Moon Dev agent integration
interface EnhancedSignal {
  // Original signal data
  signal: TradingViewSignal;
  
  // Agent analysis results
  chartAnalysis?: ChartAnalysisResult;
  riskCheck?: RiskCheckResult;
  swarmConsensus?: SwarmConsensusResult;
  
  // Final decision
  decision: 'EXECUTE' | 'REJECT' | 'MANUAL_REVIEW';
  confidence: number;
  reasoning: string;
}
```

---

## Configuration

### Environment Variables
```env
# Moon Dev Agent Configuration
OPENAI_KEY=your_openai_key
ANTHROPIC_KEY=your_anthropic_key
DEEPSEEK_KEY=your_deepseek_key

# Risk Management
MAX_DAILY_LOSS_PERCENT=5
MAX_POSITION_SIZE_PERCENT=10
MAX_LEVERAGE=5

# Swarm Configuration
SWARM_TIMEOUT_SECONDS=120
CONSENSUS_MODEL=deepseek-chat
```

### Agent Settings
```json
{
  "trading_agent": {
    "enabled": true,
    "auto_execute": false,
    "require_confirmation": true
  },
  "chart_agent": {
    "enabled": true,
    "analyze_on_signal": true,
    "min_confidence": 0.7
  },
  "risk_agent": {
    "enabled": true,
    "enforce_limits": true,
    "allow_overrides": true
  },
  "swarm_agent": {
    "enabled": true,
    "models": ["deepseek", "claude", "grok"],
    "require_consensus": true
  }
}
```

---

## Implementation Roadmap

### Phase 1: Basic Integration
- [ ] Connect webhook to agent pipeline
- [ ] Implement Chart Analysis Agent calls
- [ ] Display analysis results on dashboard

### Phase 2: Risk Management
- [ ] Integrate Risk Agent for position monitoring
- [ ] Implement limit enforcement
- [ ] Add override request flow

### Phase 3: Consensus Trading
- [ ] Enable Swarm Agent for major decisions
- [ ] Implement confidence thresholds
- [ ] Add manual review queue

### Phase 4: Full Automation
- [ ] Auto-execution with agent approval
- [ ] Real-time position management
- [ ] Trailing stop automation

---

## API Endpoints

### Agent Analysis
```
POST /api/agent/analyze
{
  "signal": { ... },
  "agents": ["chart", "risk", "swarm"]
}

Response:
{
  "chartAnalysis": { ... },
  "riskCheck": { ... },
  "swarmConsensus": { ... },
  "finalDecision": "EXECUTE",
  "confidence": 0.85
}
```

### Risk Check
```
POST /api/agent/risk-check
{
  "symbol": "BTCUSDT",
  "side": "long",
  "size": 0.1,
  "leverage": 5
}

Response:
{
  "approved": true,
  "warnings": [],
  "currentExposure": 0.15,
  "maxExposure": 0.5
}
```

### Swarm Query
```
POST /api/agent/swarm
{
  "query": "Analyze BTC setup with green dot on 4H",
  "context": { ... }
}

Response:
{
  "consensus": "Bullish setup confirmed...",
  "individualResponses": { ... },
  "confidence": 0.82
}
```
