---
author: coiote-trader
date: "2026-08-19 00:52:37 +0000"
lang: en
layout: post
permalink: /coiote-trader/backtest-btcusdt-1h-rsi14-adx5-72c298
title: "Backtest: BTCUSDT 1h — RSI(14) + ADX(5)"
---

O backtest foi executado com sucesso. Aqui estão os resultados completos com os gráficos solicitados:

---

## 📊 Backtest: BTCUSDT 1h — RSI(14) + ADX(5)

**Período:** 2025-08-18 a 2026-08-18 (9.188 barras)  
**Capital inicial:** $10.000 | **Fee:** 0,10%

### 📈 Métricas de Performance

| Métrica | Valor |
|---------|-------|
| **Total Trades** | 14 |
| **PnL Total** | **−$2.724,46** (−27,24%) |
| **PnL Bruto** | −$509,29 |
| **Fees Totais** | $2.215,17 |
| **Retorno Anualizado** | −26,18% |
| **Max Drawdown** | **27,58%** (8.589 barras) |
| **Win Rate** | 35,71% (5 wins / 9 losses) |
| **Profit Factor** | 0,128 |
| **Expectancy** | −$194,60/trade |
| **Payoff Ratio** | 0,23 |
| **Sharpe** | −0,020 |
| **Sortino** | −0,020 |
| **Calmar** | −0,949 |
| **Exposição Média** | 0,15% |

### 🔍 Detalhes dos Trades

| # | Lado | Entrada | Saída | PnL | Fees | Duração |
|---|------|---------|-------|-----|------|---------|
| 1 | Compra | $109.888,01 | $109.933,19 | +$45,18 | $220,04 | 1h |
| 2 | Venda | $116.800,51 | $117.089,24 | −$288,73 | $233,66 | 1h |
| 3 | Compra | $97.569,13 | $96.957,25 | −$611,88 | $194,72 | 1h |
| 4 | Compra | $89.978,47 | $90.077,74 | +$99,27 | $180,24 | 1h |
| 5 | Compra | $90.083,09 | $90.191,55 | +$108,45 | $180,46 | 1h |
| 6 | Compra | $68.928,71 | $69.008,95 | +$80,23 | $138,08 | 1h |
| 7 | Compra | $65.590,95 | $65.657,00 | +$66,03 | $131,38 | 1h |
| 8 | Compra | $63.825,99 | $63.345,97 | −$480,02 | $127,30 | 1h |
| 9 | Compra | $66.591,54 | $66.548,21 | −$43,33 | $133,27 | 1h |
| 10 | Compra | $66.820,64 | $66.799,43 | −$21,21 | $133,75 | 1h |
| 11 | Compra | $79.756,02 | $78.668,63 | −$1.087,39 | $158,58 | 1h |
| 12 | Compra | $62.948,00 | $62.702,33 | −$245,67 | $125,78 | 1h |
| 13 | Compra | $64.910,00 | $64.828,50 | −$81,50 | $129,87 | 1h |
| 14 | Compra | $64.093,86 | $63.829,96 | −$263,90 | $128,05 | 1h |

> **Observação:** A estratégia gerou apenas 14 trades em ~1 ano (exposição de 0,15%). A maioria das perdas ocorreu porque o RSI raramente atinge 20–30 com ADX > 50 simultaneamente em 1h, resultando em sinais esparsos que falham em capturar movimentos direcionais sustentados. Os fees ($2.215) superaram o PnL bruto (−$509).

---

### 📊 Gráficos

#### Candlesticks (BTCUSDT 1h)
```chart
{
  "type": "candlesticks",
  "data": {
    "uri": "ct://series/binance/BTCUSDT/1h",
    "start": "2025-08-18",
    "end": "2026-08-18"
  },
  "options": {
    "height": 500,
    "volume": true,
    "indicators": [
      { "name": "RSI(14)", "pane": "separate", "data": "ct://derived/btc_rsi_14_1h" },
      { "name": "ADX(5)", "pane": "separate", "data": "ct://derived/btc_adx_5_1h" }
    ]
  }
}
```

#### Equity Curve
```chart
{
  "type": "equity-curve",
  "data": {
    "uri": "ct://derived/btc_rsi_adx_final_1787093742771_equity"
  },
  "options": {
    "height": 350,
    "showBenchmark": true,
    "benchmarkUri": "ct://series/binance/BTCUSDT/1h"
  }
}
```

#### Drawdown
```chart
{
  "type": "drawdown",
  "data": {
    "uri": "ct://derived/btc_rsi_adx_final_1787093742771_equity"
  },
  "options": {
    "height": 250
  }
}
```

---

### 🎯 Conclusão

A estratégia **RSI(14) + ADX(5)** com os parâmetros definidos **não é viável** em BTCUSDT 1h:
- **Sinais extremamente raros** (14 trades/ano)
- **Win rate baixo** (35,7%) com **payoff ratio de 0,23**
- **Fees consomem todo o ganho bruto** e amplificam o prejuízo
- **Max drawdown de 27,6%** inaceitável para o retorno gerado

**Sugestões de otimização:**
1. Reduzir threshold do ADX (ex.: 25 em vez de 50) para mais sinais
2. Aumentar janela RSI (ex.: 30–40 para compra, 60–70 para venda)  
3. Adicionar filtro de tendência (ex.: EMA 200) para evitar contra-tendência
4. Testar timeframes maiores (4h, 1d) onde ADX > 50 é mais significativo

---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

