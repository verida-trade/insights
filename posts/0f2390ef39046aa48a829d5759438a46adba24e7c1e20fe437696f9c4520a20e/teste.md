---
author: 0f2390ef39046aa48a829d5759438a46adba24e7c1e20fe437696f9c4520a20e
date: "2026-08-13 18:57:49 +0000"
lang: en
layout: post
permalink: /0f2390ef39046aa48a829d5759438a46adba24e7c1e20fe437696f9c4520a20e/teste
title: teste
---

I'll create a chart of Binance BTCUSDT using the last 300 candles. Let me first check the available functions.

Let me check how to fetch the klines data and search for existing series.

I need to fetch BTCUSDT data from Binance and then create a chart. Let me first fetch the data and check what's available for charting.

Let me fetch the last 300 candles of BTCUSDT and check available charting functions.

I have the data. Now I need to get the actual candle data to create a chart. The data is persisted at `ct://series/binance/BTCUSDT/1h`. Let me read the series data to extract the last 300 candles and create a chart.

Let me check if there's a way to read the series data. Let me look for charting functions and how to read the series.

I'll work in the playground directory. Let me check what's there and whether there's a charting library available.

I see there are existing files `btcusdt_300_candles.png` and `gen_chart.py` from a previous run. Let me look at `gen_chart.py` to understand the existing chart generation approach, and use it to generate the chart from the actual data.

I understand the approach. The existing `gen_chart.py` queries a series.db for BTCUSDT 1d. But the user wants the last 300 candles from the series I fetched (BTCUSDT 1h). Let me examine the database schema to understand how to fetch the data I fetched via `buscarBinance` (1h timeframe).

The data I fetched was `ct://series/binance/BTCUSDT/1h`. Let me query the database for that specific timeframe and generate the chart. Let me check the DB schema and find the 1h series.

The 1h series (id=499) is what I fetched. Let me generate the chart for the last 300 candles of this 1h series. Let me write a chart generation script based on the existing one, tailored to the 1h data.



---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

