# Zerodha Trading Bot Plan

## Summary

Build a separate Python trading service that integrates with Zerodha Kite Connect for NIFTY options, while OpenClaw/Rakshatha becomes the Telegram control and reporting layer. The bot starts in paper mode, uses bounded auto-tuning only, and can move to live trading later under hard risk caps.

V1 must not let an LLM directly decide trades. The LLM may explain, summarize, and suggest parameter changes, but execution must pass deterministic strategy, risk, and compliance gates.

## Key Changes

- Add a new `trading/` service with Kite auth, instrument sync, market data ingestion, strategy execution, risk checks, order handling, and audit logging.
- Use Kite Connect directly for live execution; optionally use Zerodha's Kite MCP server read-only for portfolio and market inspection.
- Store every signal, decision, order, fill, P&L event, parameter change, and bot state transition in a local SQLite audit database.
- Integrate with OpenClaw/Rakshatha through Telegram commands and daily reports.
- Keep secrets out of git; use environment variables or a local secrets file with restrictive permissions.

## Trading Behavior

- Market scope: NIFTY index options only, NFO segment.
- Rollout: paper trading first, then live only after acceptance metrics pass.
- Strategy v1: deterministic NIFTY momentum breakout.
- Entry logic: build the first 15-minute NIFTY range after market open, then enter ATM or near-ATM CE on confirmed upside breakout or PE on confirmed downside breakout.
- Positioning: max one open position at a time, max three live lots after the first live smoke day.
- Exit logic: stop-loss, target, trailing stop, signal invalidation, forced intraday square-off, or risk-engine kill switch.
- Order verification: reconcile every placed order against Kite order history because API placement does not guarantee exchange execution.

## Risk And Controls

- V1 risk profile: moderate.
- Max daily loss: INR 5,000 realized plus unrealized.
- First live day: one lot max even though the eventual cap is three lots.
- Max completed trades per day: two.
- Auto-disable for the day on daily-loss breach, repeated rejected orders, stale data, token/auth failure, WebSocket failure, or broker/order API instability.
- Telegram controls: `status`, `pause`, `resume`, `paper`, `live`, `positions`, `pnl`, `closeall`, `risk`, and `why`.
- Live mode must require an explicit config switch, not only a Telegram chat command.
- The bot is for the owner's own Zerodha account only, not for managing other people's money.

## Kite And Compliance Setup

- Prerequisites: active Zerodha account, F&O enabled, Kite Connect developer app, API key, API secret, redirect URL, and static-IP VPS.
- Implement daily login/request-token exchange and access-token storage.
- Fetch the Kite instrument dump daily; do not hardcode option symbols, lot size, expiry, or instrument tokens.
- Assume SEBI retail algo framework requirements matter from April 1, 2026, including broker readiness, static IP, auditability, and operational controls.

## Test Plan

- Unit-test auth parsing, instrument selection, strategy signals, risk rejections, order reconciliation, and Telegram command parsing.
- Backtest/replay historical NIFTY candles and simulated option prices with no lookahead bias.
- Paper trade for at least 20 market sessions or equivalent replay days before live mode.
- Require zero risk-rule violations and complete audit reconciliation before live enablement.
- Live smoke must verify `pause`, `closeall`, and kill-switch behavior before normal live operation.

## References

- Kite Connect docs: https://kite.trade/docs/connect/v3/
- Kite orders: https://kite.trade/docs/connect/v3/orders/
- Kite WebSocket streaming: https://kite.trade/docs/connect/v3/websocket/
- Zerodha Kite MCP server: https://github.com/zerodha/kite-mcp-server
- SEBI Sep 30, 2025 algo framework extension: https://www.sebi.gov.in/sebi_data/attachdocs/sep-2025/1759232056254.pdf
