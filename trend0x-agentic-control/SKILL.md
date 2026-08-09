---
name: trend0x-agentic-control
description: Advanced Agential Interface for Trend0x Trading Terminal. Enables secure autonomous strategy management, risk monitoring, and UNIFIED trade execution across all connected exchanges (Bitunix, Bybit, Binance, Gate, Hyperliquid).
metadata:
  openclaw: { "emoji": "🤖", "protocol": "mcp-v1", "requires": { "auth": "bearer" } }
---

# 🤖 Trend0X Agential Control (MCP)

This document provides the necessary agential context and tool definitions for autonomous operation of the **Trend0X Terminal**. You have full authority to monitor balances, manage strategies, and execute trades on behalf of the supervisor.

## 🔑 Your Agential Credentials
> [!IMPORTANT]
> Use the following Bearer token in the `Authorization` header for all requests. 
> Keep this token secure. It provides full control over your terminal instance.

**Your Agent Token:**
`Bearer {{AGENT_TOKEN}}`

---

## 🛠️ MCP Toolset (Unified API)
All tools are accessible via the **Unified MCP Endpoint**: `https://app.trend0x.com/api/mcp`

For Claude Desktop and other SSE MCP clients, use:

```json
{
  "mcpServers": {
    "trend0x": {
      "command": "npx",
      "args": ["-y", "supergateway", "--sse", "https://app.trend0x.com/api/mcp?token=YOUR_AGENT_TOKEN"]
    }
  }
}
```

### 1. `get_market_data`
Retrieves real-time price and 24h statistics for any crypto or tradfi symbol.
- **Inputs**: `symbol` (e.g., BTCUSDT), `exchange` (optional).

### 2. `execute_trade`
Executes market orders on connected exchanges (Bybit, Bitunix, Alpaca).
- **Inputs**: `symbol`, `side` (BUY/SELL), `qty`, `exchange`, `leverage`.

### 3. `fetch_terminal_state`
Comprehensive snapshot of balances, open positions, and news.
- **Endpoint**: `GET /api/agent/unified-state`

### 4. `manage_strategies`
List or toggle trading algorithms.
- **Endpoint**: `GET /api/strategies`

### 5. `implement_strategy`
Deploy new TypeScript trading modules directly to the terminal.
- **Endpoint**: `POST /api/strategies/implement`

### 6. `manage_tasks`
Orchestrate autonomous agent workflows.
- **Endpoint**: `GET /api/tasks` | `POST /api/tasks`

### 7. `run_backtest`
Run the deterministic strategy-draft backtest before enabling live execution.

```json
{
  "symbols": ["BTCUSDT", "ETHUSDT"],
  "timeframe": "15m",
  "confirmationTimeframes": ["1h", "4h"],
  "days": 90,
  "draft": {
    "id": "my-strategy",
    "name": "Multi-timeframe momentum",
    "entryRules": [
      {"indicator": "EMA", "params": {"fast": 9, "slow": 21}, "timeframe": "15m"},
      {"indicator": "ADX", "params": {"threshold": 20}, "timeframe": "1h"}
    ],
    "filters": ["EMA(200) slope", "relative volume"],
    "confluence": 2,
    "exitRules": [
      {"indicator": "Take Profit", "params": {"percent": 3}},
      {"indicator": "Stop Loss", "params": {"percent": 1.5}}
    ]
  }
}
```

The backtest requires at least 220 valid candles, enters on the next candle
open, includes fees and slippage, and uses `STOP_FIRST` when a candle touches
both stop and target. It reports net PnL, win rate, drawdown, profit factor,
Sharpe, fees, volatility regime, and per-trade attribution.

Supported executable indicators are EMA, SMA, RSI, Volume, Relative Volume/RVOL,
VWAP, ADX, Bollinger/Bollinger Width, Price breakout, ATR, MACD, LRC, OBV,
STOCH/STOCHASTIC, and STOCHRSI. Rule-level `timeframe` (or
`params.timeframe`) is aligned to the latest completed candle in that series;
never mix an unfinished higher-timeframe candle into a signal.

### 8. Scoped risk and trade attribution

Resolve risk for every order using the exact context:

`user → exchange → symbol → side → timeframe → strategyId`

Use explicit request values only when the user intentionally overrides the
stored settings. Otherwise use the profile’s scoped values for TP, SL, leverage,
amount, trailing distance, ATR stop multiplier, and reward/risk. Generic
defaults are fallback-only.

Every live or paper execution must carry and persist:

`strategyId`, `signalId`, `timeframe`, exchange, side, strategy family, signal
source, risk profile, leverage, TP, SL, and attribution version.

Do not claim a strategy result without these fields. Group performance by
`strategyId + symbol + timeframe + exchange + side` and count only realized
closes for win rate.

### 9. Official Bybit V5 Skills
Trend0X now exposes a practical subset of the official Bybit skill repository through MCP. The underlying Bybit skill source is `https://github.com/bybit-exchange/skills`.

**Read-only market tools**
- `bybit_market_tickers`: V5 ticker data for `spot`, `linear`, `inverse`, or `option`.
- `bybit_market_kline`: V5 candle history. Use Bybit intervals such as `1`, `5`, `15`, `60`, `240`, `D`, `W`, `M`.
- `bybit_orderbook`: V5 order book snapshot.
- `bybit_instruments_info`: symbol filters, tick sizes, lot sizes, and instrument status.

**Authenticated read tools**
- `bybit_wallet_balance`: Unified account wallet balances.
- `bybit_positions`: open derivatives positions.
- `bybit_open_orders`: live open orders.
- `bybit_order_history`: historical order list.
- `bybit_trade_history`: execution/trade fills.

**Confirmed live action tools**
- `bybit_place_order`
- `bybit_cancel_order`
- `bybit_set_trading_stop`
- `bybit_set_leverage`

Live action tools must include `confirm: true`. Use `dryRun: true` to preview the request payload without sending it to Bybit.

Recommended Bybit API permissions: Read + Trade only, no Withdraw, IP whitelist enabled where possible, and dedicated AI subaccounts for autonomous agents.

### 10. `implement_strategy`
Deploy a new TypeScript strategy module to the terminal.
```bash
curl -X POST "https://app.trend0x.com/api/strategies/implement" \
     -H "Authorization: Bearer {{AGENT_TOKEN}}" \
     -H "Content-Type: application/json" \
     -d '{
       "id": "my-new-strategy",
       "name": "AI Momentum Pro",
       "code": "// TypeScript code here..."
     }'
```

### 11. `manage_tasks`
Create or list autonomous agent tasks (workflows).
**List all:**
```bash
curl -X GET "https://app.trend0x.com/api/tasks" \
     -H "Authorization: Bearer {{AGENT_TOKEN}}"
```

**Create Task:**
```bash
curl -X POST "https://app.trend0x.com/api/tasks" \
     -H "Authorization: Bearer {{AGENT_TOKEN}}" \
     -H "Content-Type: application/json" \
     -d '{
       "prompt": "Monitor ETH for RSI divergence on 15m",
       "type": "SCAN",
       "label": "ETH Monitor"
     }'
```

---

## 📱 Telegram Connectivity & Automation
The Trend0X Terminal is fully integrated with the **Aura Telegram Bot**.
- **Automated Alerts**: Strategies deployed via `implement_strategy` can automatically broadcast signals to Telegram.
- **Signal Execution**: Users can confirm agent-proposed trades directly via the bot's interactive menus.
- **Direct Interaction**: Use the `/ask` and `/task` commands in Telegram to trigger agential reasoning.

---

## 🛡️ Risk Management Guidelines
As an AI Agent, you MUST adhere to the following constraints:
1. **Interactive Verification**: For all new trades, confirm parameters (Exchange, Symbol, SL, TP) with the user before execution.
2. **Max Trade Size**: Propose size based on 2% risk-per-trade logic.
3. **Strategy Integrity**: Do not disable "Safety" or "StopLoss" strategies.

---

## 🏗️ Strategy Implementation
When designing or updating trading strategies, you MUST follow the standardized [Strategy Implementation Workflow](./STRATEGY_IMPLEMENTATION_WORKFLOW.md).
- **Pattern**: Functional Logic (Object-based).
- **Imports**: Relative paths only (`../../../types`).
- **Standard**: No imports from `@trendox/app`.

---

*Generated by AURA Intelligence Core v3.0 (Kimi/Qwen Enhanced)*
