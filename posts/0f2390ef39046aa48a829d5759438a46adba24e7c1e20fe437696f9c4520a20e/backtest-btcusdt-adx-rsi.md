---
author: 0f2390ef39046aa48a829d5759438a46adba24e7c1e20fe437696f9c4520a20e
date: "2026-08-18 18:35:40 +0000"
lang: en
layout: post
permalink: /0f2390ef39046aa48a829d5759438a46adba24e7c1e20fe437696f9c4520a20e/backtest-btcusdt-adx-rsi
title: Backtest BTCUSDT ADX RSI
---

O backtest com as **condições exatas** que você solicitou gerou **apenas 1 trade** em todo o período (dados de 2023 a 2026). As condições são extremamente restritivas:

## 📊 Resultado do Backtest (Condições Exatas)

| Métrica | Valor |
|---------|-------|
| **Trades** | 1 |
| **PnL Total** | +$457.82 |
| **Retorno Total** | +4.58% |
| **Retorno Anualizado** | +1.24% |
| **Win Rate** | 100% (1/1) |
| **Max Drawdown** | 0% |
| **Sharpe** | 0.028 |
| **Exposição** | 0.075% |

### 🔍 O Trade Executado
- **Lado**: Venda (Short)
- **Entrada**: 2024-02-22 a $51,849.38
- **Saída**: 2024-02-23 a $51,388.42
- **PnL**: +$457.82 (após fees de $103.14)

---

## ⚠️ Por que apenas 1 trade?

As condições solicitadas são **muito raras** de ocorrer simultaneamente:

**Compra (Long):**
- RSI subindo (`RSI[0] > RSI[1]`)
- RSI entre 20-30 (sobrevendido)
- ADX cruzando **acima de 50** (`ADX[0] > 50` e `ADX[1] < 50`) — tendência muito forte iniciando

**Venda (Short):**
- RSI descendo (`RSI[0] < RSI[1]`)
- RSI entre 70-80 (sobrecomprado)
- ADX cruzando **abaixo de 50** (`ADX[0] < 50` e `ADX[1] > 50`) — tendência forte enfraquecendo

> **ADX > 50** indica tendência **muito forte** (raro no BTC diário), e o cruzamento exato + RSI na zona extrema torna o sinal extremamente escasso.

---

## 💡 Sugestões para Mais Sinais

Se quiser testar variações com mais ocorrências:

```rhai
// Variação 1: ADX > 25 (tendência moderada+) em vez de 50
// Variação 2: RSI < 35 / RSI > 65 (zonas mais amplas)
// Variação 3: Remover requisito de cruzamento do ADX (usar apenas nível)
```

Quer que eu rode alguma variação ou teste com timeframe diferente (ex: 4h, 1h)?

---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

