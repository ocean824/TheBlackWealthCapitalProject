# Ultimate JSON Schema

This is the **"one JSON to rule them all"** schema for TradingView webhook alerts. It supports all trading operations through a single endpoint with a unified structure.

## Core Principles

1. **One Endpoint:** `https://yourdomain.com/api/signals/receive`
2. **One JSON Schema:** Different actions via `"event": "ENTRY" | "EXIT" | "UPDATE" | "CANCEL" | "HEARTBEAT"`
3. **Idempotency:** Every signal carries a unique `idempotency_key` to prevent duplicate trades

## Complete Schema

```json
{
  "v": "1.0",
  "source": "tradingview",
  "secret": "YOUR_WEBHOOK_SECRET",

  "strategy": {
    "name": "Ø-MASTER-CONFLUENCE",
    "strategy_id": "OMNI_001",
    "signal_id": "AUTO",
    "mode": "hybrid",
    "notes": "master script + secondary confirmations"
  },

  "market": {
    "symbol": "{{exchange}}:{{ticker}}",
    "tf": "AUTO",
    "session": "AUTO"
  },

  "event": "ENTRY",
  "side": "BUY",
  "intent": "OPEN",

  "timing": {
    "bar_time_utc": "AUTO",
    "bar_index": "AUTO",
    "calc_on": "close",
    "cooldown_sec": 120
  },

  "execution": {
    "venue": "AUTO",
    "account": "AUTO",
    "order_type": "MARKET",
    "time_in_force": "GTC",
    "reduce_only": false,
    "post_only": false,
    "client_order_id": "AUTO",
    "idempotency_key": "AUTO"
  },

  "risk": {
    "risk_model": "RISK_PERCENT",
    "risk_percent": 0.50,
    "max_daily_loss_percent": 3.0,
    "max_open_positions": 1,
    "allow_adds": false,
    "max_spread_pips": "AUTO",
    "max_slippage": "AUTO"
  },

  "position": {
    "size_model": "AUTO",
    "qty": "AUTO",
    "leverage": "AUTO",
    "entry_price_hint": "AUTO"
  },

  "bracket": {
    "sl": {
      "model": "STRUCTURE",
      "price": "AUTO",
      "distance": "AUTO",
      "hard": true
    },
    "tp": {
      "model": "RR",
      "rr": 2.0,
      "price": "AUTO",
      "partials": [
        { "rr": 1.0, "close_percent": 50 },
        { "rr": 2.0, "close_percent": 50 }
      ]
    },
    "trail": {
      "enabled": true,
      "activate_at_rr": 1.0,
      "model": "ATR",
      "atr_len": 14,
      "atr_mult": 1.5,
      "step": "AUTO"
    },
    "breakeven": {
      "enabled": true,
      "at_rr": 1.0,
      "plus": "AUTO"
    }
  },

  "confluence": {
    "score": "AUTO",
    "threshold": 70,
    "signals": {
      "trend": "AUTO",
      "momentum": "AUTO",
      "volatility": "AUTO",
      "structure": "AUTO",
      "volume": "AUTO"
    },
    "reasons": ["AUTO"]
  },

  "state": {
    "position_key": "AUTO",
    "trade_tag": "AUTO"
  }
}
```

## Event Types

### 1. ENTRY (Open New Position)

Use when confluence score passes threshold and you're flat.

```json
{
  "v": "1.0",
  "source": "tradingview",
  "secret": "...",
  "strategy": {
    "strategy_id": "OMNI_001",
    "signal_id": "LONG_A1",
    "name": "Ø-MASTER-CONFLUENCE",
    "mode": "hybrid"
  },
  "market": {
    "symbol": "{{exchange}}:{{ticker}}",
    "tf": "5"
  },
  "event": "ENTRY",
  "side": "BUY",
  "intent": "OPEN",
  "execution": {
    "order_type": "MARKET",
    "client_order_id": "OMNI_001_LONG_A1",
    "idempotency_key": "{{ticker}}_{{interval}}_{{time}}_LONG_A1"
  },
  "risk": {
    "risk_model": "RISK_PERCENT",
    "risk_percent": 0.5,
    "max_open_positions": 1
  },
  "bracket": {
    "sl": { "model": "STRUCTURE" },
    "tp": { "model": "RR", "rr": 2.0 },
    "trail": {
      "enabled": true,
      "model": "ATR",
      "atr_len": 14,
      "atr_mult": 1.5
    }
  }
}
```

### 2. UPDATE (Modify SL/TP/Trail)

Use when trade is live and your script decides to move SL or arm trailing.

```json
{
  "v": "1.0",
  "source": "tradingview",
  "secret": "...",
  "strategy": {
    "strategy_id": "OMNI_001",
    "signal_id": "MANAGE_T1"
  },
  "market": {
    "symbol": "{{exchange}}:{{ticker}}",
    "tf": "5"
  },
  "event": "UPDATE",
  "side": "BUY",
  "intent": "MODIFY",
  "state": { "position_key": "AUTO" },
  "bracket": {
    "sl": { "model": "PRICE", "price": "AUTO", "hard": true },
    "trail": {
      "enabled": true,
      "activate_at_rr": 1.0,
      "model": "ATR",
      "atr_len": 14,
      "atr_mult": 1.2
    }
  }
}
```

### 3. EXIT (Close Position)

Use when your exit condition triggers.

```json
{
  "v": "1.0",
  "source": "tradingview",
  "secret": "...",
  "strategy": {
    "strategy_id": "OMNI_001",
    "signal_id": "EXIT_X1"
  },
  "market": {
    "symbol": "{{exchange}}:{{ticker}}",
    "tf": "5"
  },
  "event": "EXIT",
  "side": "BUY",
  "intent": "CLOSE",
  "execution": { "reduce_only": true },
  "confluence": {
    "reasons": ["Exit condition met: trend invalidated"]
  }
}
```

## Alert Spam Prevention

Inside the Pine master script, enforce these rules:

### 1. Once Per Bar Close
Only fire alerts on bar close to prevent intra-bar noise.

### 2. Cooldown Window
If an alert fired in the last `cooldown_sec`, suppress duplicates.

### 3. State Machine
Track position state:
- `FLAT` → can send ENTRY
- `IN_TRADE` → can send UPDATE/EXIT only
- `LOCKOUT` (after stop or loss) → no entries for X bars

### 4. Update Throttling
SL/TP updates happen only when:
- Price moves at least X ticks/pips, OR
- RR crosses milestones (0.5R, 1R, 1.5R), OR
- Structure changes (new swing formed)

This keeps you under TradingView alert limits while still managing trades like a machine.

## Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `v` | string | Schema version |
| `source` | string | Always "tradingview" |
| `secret` | string | Webhook authentication secret |
| `event` | enum | ENTRY, UPDATE, EXIT, CANCEL, HEARTBEAT |
| `side` | enum | BUY, SELL |
| `intent` | enum | OPEN, CLOSE, MODIFY |
| `risk_model` | enum | RISK_PERCENT, FIXED_QTY, FIXED_USD |
| `sl.model` | enum | STRUCTURE, PRICE, ATR, PERCENT |
| `tp.model` | enum | RR, PRICE, PERCENT |
| `trail.model` | enum | ATR, STRUCTURE, STEP |

## TradingView Placeholders

Use these in your alert message:

| Placeholder | Description |
|-------------|-------------|
| `{{ticker}}` | Symbol name (e.g., BTCUSD) |
| `{{exchange}}` | Exchange name (e.g., BINANCE) |
| `{{interval}}` | Timeframe (e.g., 5, 15, 60) |
| `{{time}}` | Bar timestamp |
| `{{close}}` | Close price |
| `{{open}}` | Open price |
| `{{high}}` | High price |
| `{{low}}` | Low price |
| `{{volume}}` | Volume |
