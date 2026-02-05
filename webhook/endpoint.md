# Webhook Endpoint Documentation

This document explains how to set up and use the HTTPS webhook endpoint to receive TradingView alerts.

## Overview

The webhook endpoint receives alerts from TradingView and processes them for agent analysis. Signals are stored and made available for AI agents to analyze and generate trade recommendations.

## Endpoint Specification

### Receive Signal

```
POST /api/signals
Content-Type: application/json
```

**Request Body:**
```json
{
  "symbol": "BTCUSD",
  "exchange": "BINANCE",
  "timeframe": "1H",
  "signal": "long",
  "indicator": "market_cipher",
  "price": 45000.00,
  "timestamp": "2025-02-04T12:00:00Z",
  "confluence": {
    "count": 5,
    "signals": ["blue_dot", "money_flow_green", "rsi_oversold", "bullish_divergence", "support_zone"]
  },
  "metadata": {
    "rsi": 28.5,
    "volume": 1500000,
    "atr": 850.25
  }
}
```

**Response:**
```json
{
  "status": "received",
  "signal_id": "sig_abc123def456",
  "processed_at": "2025-02-04T12:00:01Z"
}
```

### Get Latest Signals

```
GET /api/signals?limit=10&symbol=BTCUSD
```

**Response:**
```json
{
  "signals": [
    {
      "id": "sig_abc123def456",
      "symbol": "BTCUSD",
      "signal": "long",
      "price": 45000.00,
      "timestamp": "2025-02-04T12:00:00Z",
      "confluence_count": 5
    }
  ],
  "total": 1
}
```

### Get Signal Details

```
GET /api/signals/:signal_id
```

**Response:**
```json
{
  "id": "sig_abc123def456",
  "symbol": "BTCUSD",
  "exchange": "BINANCE",
  "timeframe": "1H",
  "signal": "long",
  "indicator": "market_cipher",
  "price": 45000.00,
  "timestamp": "2025-02-04T12:00:00Z",
  "confluence": {
    "count": 5,
    "signals": ["blue_dot", "money_flow_green", "rsi_oversold", "bullish_divergence", "support_zone"]
  },
  "metadata": {
    "rsi": 28.5,
    "volume": 1500000,
    "atr": 850.25
  },
  "recommendation": {
    "action": "LONG",
    "confidence": 0.85,
    "entry": 45000,
    "stop_loss": 44000,
    "take_profit": [46000, 47000, 48000]
  }
}
```

## TradingView Alert Setup

### Step 1: Create Alert

1. Open your chart with Market Cipher or AlgoPro
2. Right-click on the indicator → Add Alert
3. Set your condition (e.g., "Blue Dot appears")

### Step 2: Configure Webhook

1. In the alert dialog, check "Webhook URL"
2. Enter your endpoint URL: `https://your-domain.com/api/signals`

### Step 3: Set Alert Message

Use this template for the alert message:

```json
{
  "symbol": "{{ticker}}",
  "exchange": "{{exchange}}",
  "timeframe": "{{interval}}",
  "signal": "long",
  "indicator": "market_cipher",
  "price": {{close}},
  "timestamp": "{{timenow}}",
  "confluence": {
    "count": 1,
    "signals": ["blue_dot"]
  },
  "metadata": {
    "open": {{open}},
    "high": {{high}},
    "low": {{low}},
    "close": {{close}},
    "volume": {{volume}}
  }
}
```

### TradingView Placeholders

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `{{ticker}}` | Symbol name | BTCUSD |
| `{{exchange}}` | Exchange name | BINANCE |
| `{{interval}}` | Timeframe | 60 (minutes) |
| `{{close}}` | Close price | 45000.00 |
| `{{open}}` | Open price | 44800.00 |
| `{{high}}` | High price | 45200.00 |
| `{{low}}` | Low price | 44700.00 |
| `{{volume}}` | Volume | 1500000 |
| `{{timenow}}` | Current time | 2025-02-04T12:00:00Z |
| `{{time}}` | Bar time | 2025-02-04T11:00:00Z |

## Alert Message Templates

### Market Cipher Blue Dot

```json
{
  "symbol": "{{ticker}}",
  "exchange": "{{exchange}}",
  "timeframe": "{{interval}}",
  "signal": "long",
  "indicator": "market_cipher",
  "signal_type": "blue_dot",
  "price": {{close}},
  "timestamp": "{{timenow}}",
  "confluence": {
    "count": 1,
    "signals": ["blue_dot"]
  }
}
```

### Market Cipher Red X

```json
{
  "symbol": "{{ticker}}",
  "exchange": "{{exchange}}",
  "timeframe": "{{interval}}",
  "signal": "short",
  "indicator": "market_cipher",
  "signal_type": "red_x",
  "price": {{close}},
  "timestamp": "{{timenow}}",
  "confluence": {
    "count": 1,
    "signals": ["red_x"]
  }
}
```

### AlgoPro QuantX Long

```json
{
  "symbol": "{{ticker}}",
  "exchange": "{{exchange}}",
  "timeframe": "{{interval}}",
  "signal": "long",
  "indicator": "algopro_quantx",
  "price": {{close}},
  "timestamp": "{{timenow}}",
  "confluence": {
    "count": 5,
    "signals": ["quantx_confluence"]
  },
  "risk": {
    "stop_loss": "{{plot_0}}",
    "take_profit": "{{plot_1}}"
  }
}
```

### AlgoPro Market Matrix Confluence

```json
{
  "symbol": "{{ticker}}",
  "exchange": "{{exchange}}",
  "timeframe": "{{interval}}",
  "signal": "{{strategy.order.action}}",
  "indicator": "algopro_market_matrix",
  "price": {{close}},
  "timestamp": "{{timenow}}",
  "confluence": {
    "count": "{{plot_0}}",
    "htf_bias": "{{plot_1}}",
    "ltf_bias": "{{plot_2}}"
  }
}
```

## Security

### Authentication

Include an API key in the request header:

```
Authorization: Bearer your-api-key-here
```

### IP Whitelisting

TradingView webhook IPs to whitelist:
- 52.89.214.238
- 34.212.75.30
- 54.218.53.128
- 52.32.178.7

### Rate Limiting

- Maximum 60 requests per minute per IP
- Maximum 1000 requests per hour per API key

## Error Handling

### Error Response Format

```json
{
  "error": {
    "code": "INVALID_SIGNAL",
    "message": "Missing required field: symbol",
    "details": {
      "field": "symbol",
      "expected": "string"
    }
  }
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| INVALID_SIGNAL | 400 | Malformed signal data |
| UNAUTHORIZED | 401 | Invalid or missing API key |
| RATE_LIMITED | 429 | Too many requests |
| SERVER_ERROR | 500 | Internal server error |

## Testing

### Test Endpoint

Use this endpoint for testing without storing signals:

```
POST /api/signals/test
```

### cURL Example

```bash
curl -X POST https://your-domain.com/api/signals \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{
    "symbol": "BTCUSD",
    "exchange": "BINANCE",
    "timeframe": "1H",
    "signal": "long",
    "indicator": "market_cipher",
    "price": 45000.00,
    "timestamp": "2025-02-04T12:00:00Z"
  }'
```

## Integration with Agents

Once signals are received, agents can:

1. **Fetch latest signals**: `GET /api/signals`
2. **Analyze against strategy**: Compare with strategy JSON
3. **Generate recommendations**: Create trade suggestions
4. **Draw on charts**: Use TradingView API to annotate

See the [Agent Integration Guide](../agent/integration.md) for details.
