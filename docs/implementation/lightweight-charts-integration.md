# TradingView Lightweight Charts Integration

## Overview

**TradingView Lightweight Charts** is a FREE, open-source charting library (Apache 2.0 license) that allows you to display financial charts on your Black Wealth Capital platform WITHOUT needing a TradingView Charting Library license.

**Key Benefits:**
- ✅ **FREE and open source** - No licensing fees
- ✅ **Lightweight** - Only 45KB gzipped
- ✅ **Performant** - HTML5 canvas rendering
- ✅ **Commercial use allowed** - Perfect for your trading platform
- ✅ **70+ technical indicators available** - Via community library
- ✅ **Compliant approach** - Display YOUR data, not scraping TradingView

---

## Resources

| Resource | Link | Description |
|----------|------|-------------|
| **Lightweight Charts** | [github.com/tradingview/lightweight-charts](https://github.com/tradingview/lightweight-charts) | Main charting library (13.7k stars) |
| **70+ Indicators Library** | [github.com/deepentropy/lightweight-charts-indicators](https://github.com/deepentropy/lightweight-charts-indicators) | Technical analysis indicators (33 stars) |
| **Awesome TradingView** | [github.com/tradingview/awesome-tradingview](https://github.com/tradingview/awesome-tradingview) | Curated list of TradingView resources (576 stars) |
| **Documentation** | [tradingview.com/lightweight-charts](https://www.tradingview.com/lightweight-charts/) | Official docs and examples |

---

## Installation

```bash
npm install lightweight-charts lightweight-charts-indicators oakscriptjs
```

---

## Architecture

### Compliant Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│  TradingView (Market Cipher/AlgoPro)                        │
│  ↓ Webhook Alerts (Official Feature)                        │
│  Your Webhook Endpoint (/api/signals/receive)               │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Free Data APIs (Real Price Data)                           │
│  • Binance API (Free)                                        │
│  • CoinGecko API (Free)                                      │
│  • Your Database (Store OHLCV)                               │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Lightweight Charts (Your Website)                           │
│  • Display price candlesticks                                │
│  • Overlay 70+ technical indicators                          │
│  • Draw entry/exit levels from webhook signals               │
│  • Annotate Market Cipher green dots, yellow X, etc.         │
└─────────────────────────────────────────────────────────────┘
```

**You're NOT scraping TradingView** - You're:
1. Receiving YOUR OWN alerts via webhooks (official feature)
2. Pulling price data from free APIs
3. Displaying everything on YOUR charts

---

## Basic Implementation

### 1. Create a Chart

```typescript
import { createChart, ColorType, CandlestickSeries } from 'lightweight-charts';

const chartContainer = document.getElementById('chart')!;
const chart = createChart(chartContainer, {
  layout: {
    background: { type: ColorType.Solid, color: '#0a0a0a' },
    textColor: '#00ff41',
  },
  width: 1200,
  height: 600,
  grid: {
    vertLines: { color: '#1a1a1a' },
    horzLines: { color: '#1a1a1a' },
  },
});

// Add candlestick series
const candlestickSeries = chart.addSeries(CandlestickSeries, {
  upColor: '#00ff41',
  downColor: '#ff0040',
  borderVisible: false,
  wickUpColor: '#00ff41',
  wickDownColor: '#ff0040',
});
```

### 2. Load Price Data from Free API

```typescript
// Fetch OHLCV data from Binance (free)
async function fetchPriceData(symbol: string, interval: string) {
  const response = await fetch(
    `https://api.binance.com/api/v3/klines?symbol=${symbol}&interval=${interval}&limit=500`
  );
  const data = await response.json();
  
  return data.map((candle: any) => ({
    time: Math.floor(candle[0] / 1000), // Convert to seconds
    open: parseFloat(candle[1]),
    high: parseFloat(candle[2]),
    low: parseFloat(candle[3]),
    close: parseFloat(candle[4]),
    volume: parseFloat(candle[5]),
  }));
}

// Load and display
const bars = await fetchPriceData('BTCUSDT', '1h');
candlestickSeries.setData(bars);
```

### 3. Add Technical Indicators (70+ Available)

```typescript
import { SMA, RSI, BollingerBands, MACD } from 'lightweight-charts-indicators';
import { LineSeries } from 'lightweight-charts';

// Add 20-period SMA overlay
const smaResult = SMA.calculate(bars, { len: 20, src: 'close' });
const smaSeries = chart.addSeries(LineSeries, {
  color: '#2962FF',
  lineWidth: 2,
  title: 'SMA 20',
});
smaSeries.setData(smaResult.plots.plot0);

// Add Bollinger Bands
const bbResult = BollingerBands.calculate(bars, { length: 20, mult: 2 });
const bbUpperSeries = chart.addSeries(LineSeries, { color: '#787B86', lineWidth: 1 });
const bbMiddleSeries = chart.addSeries(LineSeries, { color: '#FF6D00', lineWidth: 1 });
const bbLowerSeries = chart.addSeries(LineSeries, { color: '#787B86', lineWidth: 1 });
bbUpperSeries.setData(bbResult.plots.plot0);
bbMiddleSeries.setData(bbResult.plots.plot1);
bbLowerSeries.setData(bbResult.plots.plot2);
```

### 4. Display Webhook Signals as Annotations

```typescript
// When you receive a webhook signal from Market Cipher/AlgoPro
interface WebhookSignal {
  timestamp: number;
  type: 'green_dot' | 'yellow_x' | 'blood_diamond' | 'quantx_entry';
  price: number;
  symbol: string;
}

function addSignalMarker(signal: WebhookSignal) {
  const markers = [];
  
  if (signal.type === 'green_dot') {
    markers.push({
      time: signal.timestamp,
      position: 'belowBar',
      color: '#00ff41',
      shape: 'circle',
      text: '🟢 Green Dot',
    });
  } else if (signal.type === 'yellow_x') {
    markers.push({
      time: signal.timestamp,
      position: 'aboveBar',
      color: '#ffff00',
      shape: 'arrowDown',
      text: '⚠️ Yellow X',
    });
  } else if (signal.type === 'blood_diamond') {
    markers.push({
      time: signal.timestamp,
      position: 'aboveBar',
      color: '#ff0040',
      shape: 'arrowDown',
      text: '💎 Blood Diamond',
    });
  }
  
  candlestickSeries.setMarkers(markers);
}

// Listen for webhook signals
socket.on('signal', (signal: WebhookSignal) => {
  addSignalMarker(signal);
});
```

### 5. Draw Entry/Exit Levels

```typescript
import { createPriceLine } from 'lightweight-charts';

// Draw entry level from agent recommendation
function drawEntryLevel(price: number, timestamp: number) {
  candlestickSeries.createPriceLine({
    price: price,
    color: '#00ff41',
    lineWidth: 2,
    lineStyle: 2, // Dashed
    axisLabelVisible: true,
    title: 'Entry',
  });
}

// Draw stop-loss and take-profit
function drawTPSL(entry: number, stopLoss: number, takeProfit: number) {
  candlestickSeries.createPriceLine({
    price: stopLoss,
    color: '#ff0040',
    lineWidth: 2,
    lineStyle: 0, // Solid
    axisLabelVisible: true,
    title: 'Stop Loss',
  });
  
  candlestickSeries.createPriceLine({
    price: takeProfit,
    color: '#00ff41',
    lineWidth: 2,
    lineStyle: 0,
    axisLabelVisible: true,
    title: 'Take Profit',
  });
}
```

---

## Available Indicators (70+)

### Moving Averages
- SMA, EMA, WMA, RMA, DEMA, TEMA, HMA, LSMA, ALMA, VWMA, SMMA
- McGinley Dynamic, MA Cross, MA Ribbon

### Oscillators
- RSI, Stochastic, Stochastic RSI, CCI, Williams %R
- Awesome Oscillator, Chande MO, DPO, RVI, SMI Ergodic
- TSI, Woodies CCI, BB %B, Fisher Transform, Ultimate Oscillator

### Momentum
- MACD, Momentum, ROC, Balance of Power, Bull Bear Power
- Elder Force Index, Price Oscillator (PPO), Coppock Curve, TRIX

### Trend
- ADX, DMI, Ichimoku Cloud, Parabolic SAR, Supertrend
- Aroon, BBTrend, Choppiness Index, Mass Index, Vortex
- Williams Alligator, Zig Zag, Trend Strength Index, Chande Kroll Stop

### Volatility
- ATR, ADR, Standard Deviation, Historical Volatility, BB BandWidth

### Channels & Bands
- Bollinger Bands, Keltner Channels, Donchian Channels, Envelope, Median

### Volume
- OBV, MFI, PVT, Volume Oscillator, Chaikin MF
- Chaikin Oscillator, Ease of Movement, Klinger Oscillator

---

## Integration with Black Wealth Capital

### Step 1: Install Dependencies

```bash
cd /path/to/blackwealth-capital
npm install lightweight-charts lightweight-charts-indicators oakscriptjs
```

### Step 2: Create Chart Component

```typescript
// components/TradingChart.tsx
import { useEffect, useRef } from 'react';
import { createChart, ColorType, CandlestickSeries } from 'lightweight-charts';
import { SMA, RSI } from 'lightweight-charts-indicators';

export function TradingChart({ symbol }: { symbol: string }) {
  const chartContainerRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    if (!chartContainerRef.current) return;
    
    const chart = createChart(chartContainerRef.current, {
      layout: {
        background: { type: ColorType.Solid, color: '#0a0a0a' },
        textColor: '#00ff41',
      },
      width: chartContainerRef.current.clientWidth,
      height: 600,
    });
    
    const candlestickSeries = chart.addSeries(CandlestickSeries, {
      upColor: '#00ff41',
      downColor: '#ff0040',
    });
    
    // Fetch and display data
    fetchPriceData(symbol).then(bars => {
      candlestickSeries.setData(bars);
      
      // Add SMA overlay
      const smaResult = SMA.calculate(bars, { len: 20 });
      const smaSeries = chart.addSeries(LineSeries, { color: '#2962FF' });
      smaSeries.setData(smaResult.plots.plot0);
    });
    
    return () => chart.remove();
  }, [symbol]);
  
  return <div ref={chartContainerRef} />;
}
```

### Step 3: Connect to Webhook Signals

```typescript
// Listen for Market Cipher/AlgoPro webhook signals
useEffect(() => {
  const socket = io('/api/signals');
  
  socket.on('signal', (signal: WebhookSignal) => {
    // Add marker to chart
    addSignalMarker(signal);
    
    // Trigger agent analysis
    analyzeSignal(signal);
  });
  
  return () => socket.disconnect();
}, []);
```

### Step 4: Integrate with Agent API

```typescript
// When agent provides trade recommendation
async function handleAgentRecommendation(recommendation: TradeRecommendation) {
  // Draw entry, stop-loss, take-profit on chart
  drawEntryLevel(recommendation.entry, Date.now());
  drawTPSL(recommendation.entry, recommendation.stopLoss, recommendation.takeProfit);
  
  // Display confluence score
  displayConfluenceScore(recommendation.confluenceScore);
}
```

---

## Best Practices

### 1. Data Caching
Cache OHLCV data in your database to reduce API calls:

```typescript
// Fetch from DB first, then API if needed
const cachedBars = await db.getBars(symbol, interval);
if (cachedBars.length > 0) {
  candlestickSeries.setData(cachedBars);
} else {
  const freshBars = await fetchPriceData(symbol, interval);
  await db.saveBars(symbol, interval, freshBars);
  candlestickSeries.setData(freshBars);
}
```

### 2. Real-Time Updates
Use WebSocket for live price updates:

```typescript
const binanceWS = new WebSocket('wss://stream.binance.com:9443/ws/btcusdt@kline_1h');
binanceWS.onmessage = (event) => {
  const data = JSON.parse(event.data);
  const candle = {
    time: Math.floor(data.k.t / 1000),
    open: parseFloat(data.k.o),
    high: parseFloat(data.k.h),
    low: parseFloat(data.k.l),
    close: parseFloat(data.k.c),
  };
  candlestickSeries.update(candle);
};
```

### 3. Responsive Design
Make charts responsive to window resizing:

```typescript
useEffect(() => {
  const handleResize = () => {
    chart.applyOptions({
      width: chartContainerRef.current!.clientWidth,
    });
  };
  
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, [chart]);
```

---

## Compliance Notes

✅ **This approach is FULLY COMPLIANT** with TradingView's Terms of Service:

1. **You're NOT scraping** - Using official webhook alerts
2. **You're NOT embedding** - Using your own charting library
3. **You're NOT stealing data** - Pulling from free public APIs
4. **You're NOT violating licenses** - Lightweight Charts is Apache 2.0 (free)

❌ **What to AVOID:**
- Scraping TradingView website/charts
- Embedding TradingView iframes without permission
- Reverse-engineering TradingView Charting Library
- Using TradingView's proprietary data feeds without license

---

## Next Steps

1. **Install Lightweight Charts** in your Black Wealth Capital project
2. **Create a chart component** using the examples above
3. **Connect to free data APIs** (Binance, CoinGecko)
4. **Integrate webhook signals** from Market Cipher/AlgoPro
5. **Add agent annotations** for trade recommendations
6. **Deploy and test** with real signals

---

## Resources

- [Lightweight Charts Docs](https://tradingview.github.io/lightweight-charts/)
- [70+ Indicators Library](https://github.com/deepentropy/lightweight-charts-indicators)
- [Binance API Docs](https://binance-docs.github.io/apidocs/spot/en/)
- [CoinGecko API Docs](https://www.coingecko.com/en/api/documentation)
