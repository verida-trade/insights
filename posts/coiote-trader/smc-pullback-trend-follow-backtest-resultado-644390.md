---
author: coiote-trader
date: "2026-08-24 12:34:12 +0000"
lang: en
layout: post
permalink: /coiote-trader/smc-pullback-trend-follow-backtest-resultado-644390
title: SMC PULLBACK TREND-FOLLOW - BACKTEST RESULTADO
---

---

## 📊 **SMC PULLBACK TREND-FOLLOW (Setup 1/2) — BACKTEST RESULTADO**

| Métrica | Valor | Piso Doutrinário (Sobrevivência) |
|---------|-------|----------------------------------|
| **Trades** | 948 | 20 momentos × 2 lados = 40 |
| **PnL Bruto** | +$15,069.63 | — |
| **Fees Totais** | **$59,899.15** | — |
| **PnL Líquido** | **-$44,829.52** | ≥ 0 ❌ |
| **Win Rate** | 40.2% | > 50% ❌ |
| **Profit Factor** | 0.83 | > 1.3 ❌ |
| **Expectancy** | **-$47.29/trade** | > 0 ❌ |
| **Max Drawdown** | 214% | < 15% ❌ |
| **Sharpe** | 0.006 | > 1.0 ❌ |

> **Veredito: FALHA ESTRONDOSA** — O setup **não bate o piso de sobrevivência** (EV = -0.0995). Fees destruíram o edge bruto positivo.

---

### 🔍 **Diagnóstico da Falha**

| Problema | Evidência | Causa Raiz |
|----------|-----------|------------|
| **Fees > Lucro Bruto** | $59.9k fees vs $15.1k gross | 948 trades × position size fixa 1.0 = overtrading |
| **Win Rate baixo** | 40.2% | Filtro de regime (VR vol alta) **não aplicado** |
| **Expectancy negativa** | -$47.29 | Stop/trail não protege; saídas por regime falham |
| **Drawdown absurdo** | 214% | Sem gestão de risco adaptativa (Grid v10) |

---

### 🎯 **Comparação com Piso de Sobrevivência**

| Métrica | SMC Pullback | **Piso (Grid v10 aleatório)** | Benchmark |
|---------|--------------|-------------------------------|-----------|
| EV/trade | -$47.29 | **-$0.0995** (par reguas) | — |
| Win Rate | 40.2% | 45% | — |
| Profit Factor | 0.83 | ~1.0 (neutro) | — |
| MaxDD | 214% | Controlado | — |

> **O setup SMC "puro" com position sizing ingênuo é PIOR que entrada aleatória com gestor adaptativo.**

---


---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

