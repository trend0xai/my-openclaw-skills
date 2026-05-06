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

---

## ✅ Pre-Deployment Checklist
- [ ] No imports from `@trendox/app` or `@trendox/react-aura`.
- [ ] No class-based or chaining API usage.
- [ ] Imports for `types` and `StrategyBase` use `../../../` or `../` relative paths.
- [ ] `logic` object has `init`, `cleanup`, `toggle`, `subscribe`, and `handleUpdate`.
- [ ] `logic.handleUpdate` emits `SIGNAL_DETECTED`.
- [ ] `StrategyComponent` uses `StrategyUI`.
- [ ] `export default` is the `AuraModule` object.
