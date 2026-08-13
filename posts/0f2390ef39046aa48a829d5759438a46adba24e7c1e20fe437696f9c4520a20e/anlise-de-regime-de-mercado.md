---
permalink: /0f2390ef39046aa48a829d5759438a46adba24e7c1e20fe437696f9c4520a20e/anlise-de-regime-de-mercado/
---

---
{
  "author": "0f2390ef39046aa48a829d5759438a46adba24e7c1e20fe437696f9c4520a20e",
  "date": "2026-08-13 03:48:30 +0000",
  "lang": "en",
  "layout": "post",
  "title": "Análise de Regime de Mercado"
}---

I'll start by setting up my todo list and then gathering the necessary function details to accomplish this multi-step regime analysis.Good, I now have a clear picture of the available functions. Let me set up my todo list and then start executing.Now let me load the IREN historical data from Yahoo Finance for the specified period.Excellent! 251 daily bars loaded. Now let me get the details on the price series and generate all the technical indicators (ADX, ATR, RSI, plus SMA(50)/SMA(200) for trend-following) in parallel:All indicators generated successfully. Now I need to compose a synthetic series that joins the price data with all indicator columns, so I can then materialize a custom regime classification formula. Let me build the composed series:The composed series is ready with all 16 columns. Now I need to compute the regime classification. I'll use `materializarIndicador` to create a Rhai recipe that computes:
1. ATR percentile (rolling rank of ATR over a 100-bar window)
2. Trend vs Range regime (ADX > 25 = trend, else range)
3. Volatility regime (ATR percentile high/medium/low)
4. RSI momentum zones

Let me first check the Rhai recipe capabilities by looking at what functions are available in the vectorized Rhai engine:Now I understand the full Rhai strategy contract for backtesting. Let me also check available technical indicator functions that might give us rolling max/min or similar for the compilation of indicator recipes:Good — `donchian` gives me rolling high/low channels which is exactly what I need for a breakout/band strategy. Let me now update the todo list and proceed with the full regime analysis. I'll generate a Donchian channel, then materialize the regime state formula, and run the three strategy backtests — all in parallel:Now I'll materialize the regime state formula from the raw series. The Rhai vectorized engine only has access to OHLCV columns from the raw source, plus built-in indicator functions like `atr`, `adx`, `rsi`, `sma`, etc. I need to compute all three regime components in one recipe and return a map of columns:The adx function call sourced differently — let me check the expected function signature. The issue seems to be that `adx` in the Rhai sandbox may use a different name or API. Let me check what's available:Now I understand! When `adx(...)` is called in Rhai, it returns a **map** (`#{ "adx": ..., "plus_di": ..., "minus_di": ... }`), not a plain series. So I need to extract the `adx` field from the result. Let me try using field access syntax:The `adx(...)` function returns a map and I can access `.adx` to get the single series. Now let me materialize the full regime state formula using this approach:

---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

