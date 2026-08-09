# 🛡️ Strategy Implementation Workflow

This workflow defines the mandatory standards for implementing trading strategy modules in the Trend0X terminal. Adhering to these rules ensures compatibility with the Vite build system and prevents common runtime errors.

## 📁 File Structure
All strategies must be contained in a dedicated directory:
`modules/trading/trading-strategy-<unique-id>/index.tsx`

---

## 🧩 Module Pattern: Functional Logic
The terminal uses a **Functional Logic Pattern**, NOT a Class-based structure. Every strategy must export a default `AuraModule` object.

### 1. Mandatory Imports
Always use relative paths to the root types and utilities. Do NOT use aliases like `@trendox/app` or `@trendox/react-aura`.
```typescript
import React from 'react';
import { AuraModule, ModuleMetadata, IEventBus, SignalPayload, MarketUpdatePayload } from "../../../types";
import { StrategyUI, InputField } from "../StrategyBase";
import { Indicators } from "../../../utils/indicators";
```

### 2. Metadata Definition
The metadata object defines the strategy's appearance and categorization.
```typescript
export const metadata: ModuleMetadata = {
    id: "trading-strategy-example-id",
    name: "Strategy Name",
    version: "1.0.0",
    description: "Brief technical description of the logic.",
    author: "AURA AI",
    slot: "sidebar",
    order: 100,
    strategyCategory: "Scalping", // Options: Scalping, DayTrading, Swing, Investing
    suitableTimeframes: ["1m", "5m"],
    riskProfile: "Moderate",
    assetClasses: ["Crypto"]
};
```

### 3. Logic Object
The `logic` object contains the state and the `handleUpdate` loop.
```typescript
export const logic = {
    bus: null as any,
    isActive: true,
    params: { 
        rsiPeriod: 14, 
        overbought: 70, 
        oversold: 30, 
        timeframe: '5m' 
    },
    listeners: new Set<any>(),

    init(bus: any) { 
        this.bus = bus; 
    },

    cleanup() { 
        this.bus = null; 
    },

    toggle() { 
        this.isActive = !this.isActive; 
    },

    subscribe(cb: any) { 
        this.listeners.add(cb); 
        return () => this.listeners.delete(cb); 
    },

    setParameters(p: any) { 
        this.params = { ...this.params, ...p }; 
    },

    handleUpdate(payload: MarketUpdatePayload) {
        if (!this.isActive || !this.bus || !payload.candles) return;
        if (payload.timeframe !== this.params.timeframe) return;

        // Implementation of signal detection logic using Indicators
        // ...
        
        if (signalDetected) {
            const signal: SignalPayload = {
                id: crypto.randomUUID(),
                symbol: payload.symbol,
                pair: payload.symbol,
                type: 'LONG', // or 'SHORT'
                strategy: metadata.id,
                price: payload.lastPrice,
                confidence: 0.8,
                timestamp: Date.now(),
                timeframe: payload.timeframe,
                reasoning: "Description of the signal...",
                chartData: payload.candles.slice(-50)
            };
            this.bus.emit('SIGNAL_DETECTED', signal, metadata.id);
            this.listeners.forEach((cb: any) => cb(signal));
        }
    }
};
```

### 4. UI Component
Use the standardized `StrategyUI` and `InputField` for the settings panel.
```typescript
const StrategyComponent: React.FC = () => {
    return (
        <StrategyUI 
            metadata={metadata} 
            logic={logic as any} 
            icon="🧠" 
            description={metadata.description} 
            renderSettings={(params: any, handleChange: any) => (
                <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '8px' }}>
                    <InputField label="RSI Period" value={params.rsiPeriod} onChange={v => handleChange('rsiPeriod', v)} />
                    {/* Add other inputs here */}
                </div>
            )}
        />
    );
};
```

### 5. Export Wrapper
Finally, export the module using the `AuraModule` interface.
```typescript
const StrategyModule: AuraModule = {
    metadata,
    onMount: (bus: IEventBus) => logic.init(bus),
    onUnmount: () => logic.cleanup(),
    logic,
    component: StrategyComponent
};

export default StrategyModule;
```

---

## 🚫 Critical Prohibitions
1.  **NO `@trendox/app` or `@trendox/react-aura`**: These are legacy/external aliases that will fail Vite import analysis. Use `../../../types`.
2.  **NO Class Inheritance or Method Chaining**: Do NOT use `class MyStrategy extends AuraModule` or `new AuraModule().addIndicator()`. The system expects a plain object matching the `AuraModule` interface.
3.  **Relative Indicators**: Always use `Indicators.calculateXXX` from the local `utils/indicators` file to ensure calculation consistency.
4.  **Timeframe Filtering**: Always check `if (payload.timeframe !== this.params.timeframe) return;` in the `handleUpdate` to avoid processing data on unintended scales.

## 🧪 Backtest and Attribution Requirements

Before a generated strategy is promoted beyond probe mode:

1. Define every `entryRules` indicator and its numeric parameters explicitly.
   The executable draft engine supports EMA, SMA, RSI, Volume, Relative
   Volume/RVOL, VWAP, ADX, BB/Bollinger Width, PRICE, ATR, MACD, LRC, OBV,
   STOCH/STOCHASTIC, and STOCHRSI.
2. Attach `timeframe` to each rule that uses a different series from the primary
   timeframe. Request the additional series through `confirmationTimeframes`.
   The engine aligns rules to the latest completed candle and prevents
   look-ahead from an unfinished higher-timeframe candle.
3. Run `/api/backtest/run` with at least 220 candles and record the exact draft,
   filters, confluence, TP/SL, fees, slippage, leverage, sizing, and volatility
   regime. Treat same-candle stop/target collisions as `STOP_FIRST`.
4. Resolve live risk with the scoped context `user + exchange + symbol + side +
   timeframe + strategyId`. Do not replace a symbol/timeframe override with a
   profile-wide TP, SL, or leverage value.
5. Attach immutable attribution to every signal and order: `strategyId`,
   `signalId`, `timeframe`, exchange, side, strategy family, source, risk
   profile, leverage, TP, SL, and attribution version.
6. Report performance only from realized closes. Segment by strategy, pair,
   timeframe, exchange, and side; do not count open order records as wins or
   losses.

---

## ✅ Pre-Deployment Checklist
- [ ] No imports from `@trendox/app` or `@trendox/react-aura`.
- [ ] No class-based or chaining API usage.
- [ ] Imports for `types` and `StrategyBase` use `../../../` or `../` relative paths.
- [ ] `logic` object has `init`, `cleanup`, `toggle`, `subscribe`, and `handleUpdate`.
- [ ] `logic.handleUpdate` emits `SIGNAL_DETECTED`.
- [ ] `StrategyComponent` uses `StrategyUI`.
- [ ] `export default` is the `AuraModule` object.
- [ ] A deterministic draft backtest has been run with the final parameters.
- [ ] Every non-primary timeframe is declared and supplied as a confirmation series.
- [ ] Scoped risk settings are resolved for the target exchange, symbol, side, timeframe, and strategy.
- [ ] Signals and executions persist complete attribution fields.
- [ ] Performance reports count realized closes only.
