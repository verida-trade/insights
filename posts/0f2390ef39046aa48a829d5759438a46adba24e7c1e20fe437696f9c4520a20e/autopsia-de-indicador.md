---
author: 0f2390ef39046aa48a829d5759438a46adba24e7c1e20fe437696f9c4520a20e
date: "2026-08-13 18:18:17 +0000"
lang: en
layout: post
permalink: /0f2390ef39046aa48a829d5759438a46adba24e7c1e20fe437696f9c4520a20e/autopsia-de-indicador
title: Autopsia de Indicador
---

I'll start by understanding the available tools and loading the necessary historical data.

I need to understand the data functions. Let me get details on the key functions for loading historical data, RSI, and how to access series data.

I have the details I need. Let me set up my plan and understand the data access. I notice there's no direct "read raw rows" function exposed in the normal API — but I need to access actual data for RSI analysis. Let me check what functions exist to query/pull series data. Let me look at `listarSeries` and other functions, and check what data access is available. Actually, the analysis requires me to access the underlying data (RSI values, forward returns).

Let me look at the available data access. The `ctBacktest` runs strategies, `montarPipelineIndicadores` builds derived series. But to do correlation analysis I need raw data access. Let me check `Summon.load` and the summarize function, and also look at what SDK functions exist for reading raw series rows.

Let me set up the todo and start loading data while I figure out data access.

I now understand the tooling landscape. The key challenge here is that the analysis requires raw data access (RSI values, forward returns) to compute correlations and frequencies. The available SDK functions provide data manipulation, but I need direct access to the underlying time series rows.

Let me note that there's a `materializarIndicador` function that computes RSI as a derived series, and `analisarDataset` which provides EDA summaries. But for the detailed correlation analysis with forward returns, I'll need raw data.

Let me set up the todos, load the historical data, and compute RSI. Let me start by loading the BTCUSD historical data from Binance for the specified period.

Let me create my plan and start executing. First, let me set up the todo list and load the historical data.

Let me load the historical data from Binance and set up RSI computation. First, let me check what data access is available for reading raw rows since I need to compute correlations. Let me load the data and set up the RSI series.

Data loaded: 8761 rows of BTCUSDT 1h. Now let me compute RSI and understand the data access. The key challenge is I need raw data access to compute forward returns and correlations. Let me check if I can retrieve the actual series rows.

Let me get details on how to read raw data. Let me check `analisarDataset` which can compute EDA with a target. But for forward returns I need to construct derived series. Let me first compute RSI and materialize forward return indicators.

Let me set up the RSI series and build the required derived data. I'll compute RSI and forward returns using the pipeline indicator tools.



---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

