# 🔑 Trend0X Connection & API Guide

This guide explains how to connect your exchange accounts and enable autonomous trading via the Trend0X OpenClaw skills.

## 1. Connecting Exchanges
To execute trades, you must link your API keys in the Trend0X Dashboard.

### Supported Exchanges
- **Bybit**: Primary for Perpetual Futures.
- **Bitunix**: Alternative for high-leverage crypto.
- **Alpaca**: TradFi (US Stocks & Options).
- **Binance/Gate/Hyperliquid**: Integrated via Unified Bridge.

### How to add API Keys
1. Navigate to `https://demo.trend0x.com/neuralink`.
2. Select your exchange from the **Multi-Exchange Hub**.
3. Enter your **API Key**, **Secret**, and **Passphrase** (if required).
4. Ensure the keys have **Trading** and **Balance Query** permissions enabled.

## 2. Enabling Automated Signal Execution
Trend0X uses a "Strategy Module" architecture. Automated execution works as follows:

### Signal Source
Strategies generate `SignalPayload` objects when conditions are met. These signals are routed through the `neuroMesh` kernel.

### Automation Modes
- **Semi-Autonomous**: Signals are sent to the **Telegram Bot** for one-click user confirmation.
- **Full-Autonomous**: Requires a **Premium Subscription** (Agentic Task Flows). When enabled, the agent executes the signal directly using the best available liquidity.

## 3. Bot Connectivity
- **Telegram Bot**: Search for `@trend0x_agent_bot` (or your private instance).
- **Connection**: Link your account using the `/login` command.
- **Execution**: When the terminal generates a signal, you will receive a Telegram alert with "Accept" or "Reject" buttons.

## 4. MCP & Agent Network Integration
The skills in this repository follow the **Model Context Protocol (MCP)**. They are designed for:
- **Claude (Antropic)**: Full support via Desktop App or custom MCP bridge.
- **OpenClaw**: Native support for agential orchestration.

---
*Powered by AURA Intelligence Core*
