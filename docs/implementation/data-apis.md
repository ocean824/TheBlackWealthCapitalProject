# Free Data APIs Integration Guide

> **Purpose**: Integrate free and universal API keys to power the BWC platform with real market data, similar to Luxalgo AI capabilities.

---

## Available Free APIs

### Cryptocurrency Data

| API | Purpose | Rate Limit | Auth Required |
|-----|---------|------------|---------------|
| CoinGecko | Prices, market cap, historical data | 50 calls/min | No |
| Binance Public | Real-time prices, orderbook, trades | 1200 calls/min | No |
| CryptoCompare | Historical OHLCV, social stats | 100K calls/month | API Key (free) |
| Messari | Fundamental data, metrics | 20 calls/min | API Key (free) |

### Stock/Forex Data

| API | Purpose | Rate Limit | Auth Required |
|-----|---------|------------|---------------|
| Alpha Vantage | Stocks, forex, crypto | 5 calls/min | API Key (free) |
| Yahoo Finance (yfinance) | Stocks, ETFs, indices | Unofficial | No |
| Finnhub | Real-time quotes, news | 60 calls/min | API Key (free) |
| Polygon.io | Stocks, options, forex | 5 calls/min (free) | API Key (free) |

### On-Chain Data

| API | Purpose | Rate Limit | Auth Required |
|-----|---------|------------|---------------|
| Etherscan | ETH transactions, contracts | 5 calls/sec | API Key (free) |
| Solscan | SOL transactions, tokens | 100 calls/min | No |
| Whale Alert | Large transactions | Limited | API Key (free) |

### Sentiment & News

| API | Purpose | Rate Limit | Auth Required |
|-----|---------|------------|---------------|
| Fear & Greed Index | Market sentiment | Unlimited | No |
| NewsAPI | News headlines | 100 calls/day | API Key (free) |
| Reddit API | Social sentiment | 60 calls/min | OAuth |

---

## Implementation Examples

### CoinGecko Integration

```typescript
// server/services/dataProviders/coingecko.ts
import axios from "axios";

const BASE_URL = "https://api.coingecko.com/api/v3";

export interface PriceData {
  symbol: string;
  price: number;
  change24h: number;
  volume24h: number;
  marketCap: number;
  lastUpdated: Date;
}

export async function getCurrentPrice(coinId: string): Promise<PriceData> {
  const response = await axios.get(`${BASE_URL}/simple/price`, {
    params: {
      ids: coinId,
      vs_currencies: "usd",
      include_24hr_change: true,
      include_24hr_vol: true,
      include_market_cap: true,
      include_last_updated_at: true
    }
  });
  
  const data = response.data[coinId];
  return {
    symbol: coinId.toUpperCase(),
    price: data.usd,
    change24h: data.usd_24h_change,
    volume24h: data.usd_24h_vol,
    marketCap: data.usd_market_cap,
    lastUpdated: new Date(data.last_updated_at * 1000)
  };
}

export async function getHistoricalOHLCV(
  coinId: string,
  days: number = 30
): Promise<Array<{time: number; open: number; high: number; low: number; close: number}>> {
  const response = await axios.get(`${BASE_URL}/coins/${coinId}/ohlc`, {
    params: {
      vs_currency: "usd",
      days
    }
  });
  
  return response.data.map((candle: number[]) => ({
    time: candle[0],
    open: candle[1],
    high: candle[2],
    low: candle[3],
    close: candle[4]
  }));
}

export async function getMarketData(coinIds: string[]): Promise<PriceData[]> {
  const response = await axios.get(`${BASE_URL}/coins/markets`, {
    params: {
      vs_currency: "usd",
      ids: coinIds.join(","),
      order: "market_cap_desc",
      sparkline: false
    }
  });
  
  return response.data.map((coin: any) => ({
    symbol: coin.symbol.toUpperCase(),
    price: coin.current_price,
    change24h: coin.price_change_percentage_24h,
    volume24h: coin.total_volume,
    marketCap: coin.market_cap,
    lastUpdated: new Date(coin.last_updated)
  }));
}
```

### Binance Public API Integration

```typescript
// server/services/dataProviders/binance.ts
import axios from "axios";

const BASE_URL = "https://api.binance.com/api/v3";

export interface KlineData {
  openTime: number;
  open: number;
  high: number;
  low: number;
  close: number;
  volume: number;
  closeTime: number;
  quoteVolume: number;
  trades: number;
}

export async function getKlines(
  symbol: string,
  interval: string = "1h",
  limit: number = 500
): Promise<KlineData[]> {
  const response = await axios.get(`${BASE_URL}/klines`, {
    params: { symbol, interval, limit }
  });
  
  return response.data.map((k: any[]) => ({
    openTime: k[0],
    open: parseFloat(k[1]),
    high: parseFloat(k[2]),
    low: parseFloat(k[3]),
    close: parseFloat(k[4]),
    volume: parseFloat(k[5]),
    closeTime: k[6],
    quoteVolume: parseFloat(k[7]),
    trades: k[8]
  }));
}

export async function getOrderBook(
  symbol: string,
  limit: number = 100
): Promise<{ bids: [number, number][]; asks: [number, number][] }> {
  const response = await axios.get(`${BASE_URL}/depth`, {
    params: { symbol, limit }
  });
  
  return {
    bids: response.data.bids.map((b: string[]) => [parseFloat(b[0]), parseFloat(b[1])]),
    asks: response.data.asks.map((a: string[]) => [parseFloat(a[0]), parseFloat(a[1])])
  };
}

export async function get24hrTicker(symbol: string): Promise<{
  priceChange: number;
  priceChangePercent: number;
  weightedAvgPrice: number;
  lastPrice: number;
  volume: number;
  quoteVolume: number;
  highPrice: number;
  lowPrice: number;
}> {
  const response = await axios.get(`${BASE_URL}/ticker/24hr`, {
    params: { symbol }
  });
  
  return {
    priceChange: parseFloat(response.data.priceChange),
    priceChangePercent: parseFloat(response.data.priceChangePercent),
    weightedAvgPrice: parseFloat(response.data.weightedAvgPrice),
    lastPrice: parseFloat(response.data.lastPrice),
    volume: parseFloat(response.data.volume),
    quoteVolume: parseFloat(response.data.quoteVolume),
    highPrice: parseFloat(response.data.highPrice),
    lowPrice: parseFloat(response.data.lowPrice)
  };
}
```

### Fear & Greed Index

```typescript
// server/services/dataProviders/fearGreed.ts
import axios from "axios";

export interface FearGreedData {
  value: number;
  classification: string;
  timestamp: Date;
}

export async function getFearGreedIndex(): Promise<FearGreedData> {
  const response = await axios.get("https://api.alternative.me/fng/");
  const data = response.data.data[0];
  
  return {
    value: parseInt(data.value),
    classification: data.value_classification,
    timestamp: new Date(parseInt(data.timestamp) * 1000)
  };
}

export async function getFearGreedHistory(days: number = 30): Promise<FearGreedData[]> {
  const response = await axios.get(`https://api.alternative.me/fng/?limit=${days}`);
  
  return response.data.data.map((d: any) => ({
    value: parseInt(d.value),
    classification: d.value_classification,
    timestamp: new Date(parseInt(d.timestamp) * 1000)
  }));
}
```

---

## Unified Data Service

```typescript
// server/services/dataService.ts
import * as coingecko from "./dataProviders/coingecko";
import * as binance from "./dataProviders/binance";
import * as fearGreed from "./dataProviders/fearGreed";

export class DataService {
  private cache: Map<string, { data: any; expiry: number }> = new Map();
  private cacheTTL = 60000; // 1 minute default
  
  private async getCached<T>(key: string, fetcher: () => Promise<T>, ttl?: number): Promise<T> {
    const cached = this.cache.get(key);
    if (cached && cached.expiry > Date.now()) {
      return cached.data as T;
    }
    
    const data = await fetcher();
    this.cache.set(key, { data, expiry: Date.now() + (ttl || this.cacheTTL) });
    return data;
  }
  
  async getPrice(symbol: string): Promise<number> {
    // Try Binance first (faster), fallback to CoinGecko
    try {
      const ticker = await this.getCached(
        `price:${symbol}`,
        () => binance.get24hrTicker(`${symbol}USDT`)
      );
      return ticker.lastPrice;
    } catch {
      const coinId = this.symbolToCoinId(symbol);
      const data = await this.getCached(
        `price:${symbol}`,
        () => coingecko.getCurrentPrice(coinId)
      );
      return data.price;
    }
  }
  
  async getOHLCV(symbol: string, interval: string, limit: number): Promise<any[]> {
    return this.getCached(
      `ohlcv:${symbol}:${interval}:${limit}`,
      () => binance.getKlines(`${symbol}USDT`, interval, limit),
      300000 // 5 minute cache for OHLCV
    );
  }
  
  async getMarketSentiment(): Promise<{ fearGreed: number; classification: string }> {
    const data = await this.getCached(
      "sentiment:feargreed",
      () => fearGreed.getFearGreedIndex(),
      3600000 // 1 hour cache
    );
    return { fearGreed: data.value, classification: data.classification };
  }
  
  private symbolToCoinId(symbol: string): string {
    const mapping: Record<string, string> = {
      BTC: "bitcoin",
      ETH: "ethereum",
      SOL: "solana",
      // Add more mappings
    };
    return mapping[symbol.toUpperCase()] || symbol.toLowerCase();
  }
}

export const dataService = new DataService();
```

---

## Warning: API Limitations

**Important Considerations:**

1. **Rate Limits**: All free APIs have rate limits. Implement caching and request queuing.

2. **Data Freshness**: Free tier data may be delayed (15-30 minutes for some providers).

3. **Reliability**: Free APIs may have downtime. Implement fallback providers.

4. **Terms of Service**: Always check API terms before commercial use.

5. **No Financial Advice**: Display appropriate disclaimers about data accuracy.

### Recommended Disclaimers

```typescript
export const DATA_DISCLAIMER = `
Market data is provided for informational purposes only and may be delayed.
Black Wealth Capital does not guarantee the accuracy, completeness, or timeliness of the data.
This information should not be considered financial advice.
Always do your own research before making investment decisions.
`;
```

---

## MCP Data API Integration

If available through the Manus MCP, use the built-in data APIs with appropriate warnings:

```typescript
// Check if MCP data API is available
import { exec } from "child_process";

async function checkMCPDataAPI(): Promise<boolean> {
  return new Promise((resolve) => {
    exec("manus-mcp-cli tool list --server data_api", (error, stdout) => {
      resolve(!error && stdout.includes("get_price"));
    });
  });
}

// Use MCP if available, fallback to free APIs
export async function getMarketData(symbol: string) {
  const hasMCP = await checkMCPDataAPI();
  
  if (hasMCP) {
    // Use MCP data API
    const result = await exec(`manus-mcp-cli tool call get_price --server data_api --input '{"symbol":"${symbol}"}'`);
    return JSON.parse(result);
  } else {
    // Fallback to free APIs
    return dataService.getPrice(symbol);
  }
}
```
