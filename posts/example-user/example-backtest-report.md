---
layout: post
title: "Example Backtest Report"
author: "example-user"
date: "2026-08-12 19:00:00 +0000"
lang: en
asset: BTCUSD
timeframe: 1h
tags: ["backtest", "momentum", "rsi"]
---

# Example Backtest Report

This is a sample insight published via CT Lab.

## Summary

- **Asset:** BTCUSD
- **Timeframe:** 1h
- **Period:** 2024-01-01 to 2024-06-30
- **Strategy:** RSI oversold bounce with ADX trend filter

## Results

| Metric | Value |
|--------|-------|
| Net Profit | +12.4% |
| Max Drawdown | -4.2% |
| Sharpe Ratio | 1.87 |
| Win Rate | 62% |

## Strategy Code

```rhai
// Enter when RSI < 30 and ADX > 25
let rsi_val = rsi(close, 14);
let adx_val = adx(high, low, close, 14);
if rsi_val[0] < 30 && adx_val[0] > 25 {
    buy(1);
}
```

---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice.
