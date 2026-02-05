# TradingView MCP Integration Guide

This document covers the available Model Context Protocol (MCP) servers for TradingView integration and their compliance considerations.

## Available MCP Servers

### 1. atilaahmettaner/tradingview-mcp

**Repository:** https://github.com/atilaahmettaner/tradingview-mcp

**Purpose:** AI-powered market analysis using TradingView data for crypto and stock screening.

**Key Features:**

| Feature | Description |
|---------|-------------|
| Market Screening | Top gainers/losers, Bollinger Band scans |
| Technical Analysis | RSI, MACD, Bollinger Bands, ADX, Stochastic |
| Pattern Recognition | Consecutive candle detection, multi-timeframe analysis |
| Multi-Exchange | Binance, KuCoin, Bybit, NASDAQ, NYSE, BIST |

**Available Tools:**

| Tool | Description |
|------|-------------|
| `top_gainers` | Find highest performing assets |
| `top_losers` | Find biggest declining assets |
| `bollinger_scan` | Find assets with tight Bollinger Bands |
| `rating_filter` | Filter by Bollinger Band rating (-3 to +3) |
| `coin_analysis` | Complete technical analysis for any symbol |
| `consecutive_candles_scan` | Find candlestick patterns |
| `advanced_candle_pattern` | Multi-timeframe pattern analysis |

**Compliance Status:** ⚠️ **CAUTION**

This MCP server fetches data from TradingView's public endpoints. While it doesn't scrape the UI directly, it may be accessing unofficial APIs. Use at your own risk.

**Recommended Use:** Market screening and analysis only. Do not use for automated trading decisions without verification.

---

### 2. ertugrul59/tradingview-chart-mcp

**Repository:** https://github.com/ertugrul59/tradingview-chart-mcp

**Purpose:** Fetch TradingView chart images based on ticker and interval.

**Key Features:**

| Feature | Description |
|---------|-------------|
| Chart Images | Fetch chart snapshots as images |
| Browser Pooling | 70-80% faster concurrent performance |
| Performance Stats | Built-in monitoring and statistics |
| Clipboard Capture | Direct base64 image capture |

**Available Tools:**

| Tool | Description |
|------|-------------|
| `get_tradingview_chart_image` | Fetch chart snapshot image URL |
| `get_performance_stats` | Get performance statistics |

**Compliance Status:** ❌ **NOT RECOMMENDED**

This MCP server uses browser automation to scrape TradingView charts. This violates TradingView's Terms of Service and could result in account bans.

**Why It's Risky:**
- Uses Selenium/Playwright to automate browser
- Requires TradingView session cookies
- Captures screenshots from the UI
- TradingView actively detects and bans scrapers

---

## Compliance-First Architecture

Instead of relying on MCP servers that may violate ToS, we recommend this compliant architecture:

### Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMPLIANT DATA SOURCES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐    ┌──────────────────┐                   │
│  │ TradingView      │    │ Market Data      │                   │
│  │ Webhook Alerts   │    │ Provider API     │                   │
│  │ (Official)       │    │ (Licensed)       │                   │
│  └────────┬─────────┘    └────────┬─────────┘                   │
│           │                       │                              │
│           └───────────┬───────────┘                              │
│                       ▼                                          │
│              ┌──────────────────┐                                │
│              │  Your Backend    │                                │
│              │  (Manus App)     │                                │
│              └────────┬─────────┘                                │
│                       │                                          │
│           ┌───────────┼───────────┐                              │
│           ▼           ▼           ▼                              │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐                   │
│  │ Agent      │ │ Datafeed   │ │ Broker     │                   │
│  │ Analysis   │ │ Charts     │ │ API        │                   │
│  └────────────┘ └────────────┘ └────────────┘                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Compliant Alternatives

| Need | Non-Compliant Approach | Compliant Alternative |
|------|------------------------|----------------------|
| Chart images | Scrape TradingView UI | Use licensed Charting Library |
| Indicator values | Parse DOM elements | Send via webhook alerts |
| Market data | Unofficial APIs | Licensed data provider |
| Trade execution | Automate TV interface | Use broker API directly |

---

## Recommended MCP Usage

### Safe to Use

1. **atilaahmettaner/tradingview-mcp** - For market screening and analysis
   - Use for research and idea generation
   - Don't rely on it for trade execution
   - Verify signals with official sources

### Not Recommended

1. **ertugrul59/tradingview-chart-mcp** - Browser automation
   - Violates ToS
   - Risk of account ban
   - Use licensed Charting Library instead

---

## Integration with Manus

### Using tradingview-mcp with Manus

If you want to use the market screening MCP server with Manus, you can configure it as follows:

```bash
# List available tools
manus-mcp-cli tool list --server tradingview-mcp

# Call a tool
manus-mcp-cli tool call top_gainers --server tradingview-mcp --input '{"exchange": "BINANCE", "limit": 10}'
```

### Webhook-Based Integration (Recommended)

Instead of using MCP servers that may violate ToS, configure TradingView alerts to send webhooks to your Manus app:

1. **Set up webhook endpoint** in your Manus app (already done at `/api/signals/receive`)
2. **Create TradingView alert** with webhook URL
3. **Configure JSON payload** using the Ultimate JSON Schema
4. **Process signals** in your backend
5. **Display on your charts** using licensed Charting Library

This approach is 100% compliant and gives you full control over your data.

---

## Summary

| MCP Server | Compliance | Recommendation |
|------------|------------|----------------|
| tradingview-mcp | ⚠️ Caution | Use for research only |
| tradingview-chart-mcp | ❌ Risky | Avoid, use Charting Library |
| Your webhook endpoint | ✅ Compliant | Primary integration method |

**The Golden Rule:** Use official TradingView features (webhooks, licensed library) for production. Use MCP servers only for research and idea generation, never for automated trading.
