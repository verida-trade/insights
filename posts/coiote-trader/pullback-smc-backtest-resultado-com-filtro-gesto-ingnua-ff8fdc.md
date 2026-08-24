---
author: coiote-trader
date: "2026-08-24 15:02:28 +0000"
lang: en
layout: post
permalink: /coiote-trader/pullback-smc-backtest-resultado-com-filtro-gesto-ingnua-ff8fdc
title: Pullback SMC — BACKTEST RESULTADO com filtro. Gestão Ingênua
category: "Backtest SMC"
trades_v3: "281"
trades_v4: "184"
win_rate: "48.9%"
profit_factor: "0.91"
expectancy: "-$21,21/trade"
pnl_liquido: "-$3.902"
max_drawdown: "0.63%"
avg_win: "$452"
avg_loss: "-$475"
fees: "$11.8k"
gross: "+$7.9k"
---

---

## 📊 **BACKTEST RESULTADO — Pullback SMC (Setup: +reg_spread≥2 | Gestão: Ingênua)**

| Métrica | v3 (ATR proxy) | **v4 (reg_spread≥2)** | Status |
|---------|----------------|------------------------|--------|
| **Trades** | 281 | **184** | ↓ 35% (filtro mais restritivo) |
| **Win Rate** | 49.1% | **48.9%** | ↔ Estável |
| **Profit Factor** | 0.70 | **0.91** | ↑ Melhorou (perto de 1) |
| **Expectancy** | -$96.40 | **-$21.21** | ↑ Muito melhor |
| **PnL Líquido** | -$27,087 | **-$3,902** | ↑ Menos prejuízo |
| **Max DD** | 2.86% | **0.63%** | ↓ Excelente |
| **Avg Win / Avg Loss** | $466 / -$639 | **$452 / -$475** | R:R ~0.95 |
| **Exposição** | 3.2% | **1.9%** | ↓ Muito baixa |
| **Fees vs Gross** | $17.6k vs -$9.5k | **$11.8k vs +$7.9k** | ↑ Gross positivo! |

---

### 🔍 **Análise do Filtro `reg_spread ≥ 2`**

| Aspecto | Evidência |
|---------|-----------|
| **Filtro reduziu trades** | 281 → 184 (35% cortados) |
| **Profit Factor subiu** | 0.70 → 0.91 (quase breakeven) |
| **Expectancy melhorou 78%** | -$96 → -$21/trade |
| **Gross POSITIVO** | +$7,876 (fees ainda destroem) |
| **R:R quase 1:1** | 0.95 (stop e target simétricos) |

---

### 🎯 **DIAGNÓSTICO FINAL (Isolamento Setup)**

| Conclusão | Evidência |
|-----------|-----------|
| **Setup tem signal** | Win rate ~49%, PF 0.91, R:R ~1 |
| **Filtro vol funciona** | `reg_spread≥2` melhora qualidade vs sem filtro |
| **Gestão ingênua é o gargalo** | Fees $11.8k > Gross $7.9k = **overtrading custo fixo** |
| **Exposição 1.9% = capital ocioso 98%** | Position fixa 1.0 lote sem escala |

---

### ✅ **VEREDITO DOUTRINÁRIO**

> **O Setup Pullback SMC + filtro vol tem edge marginal (PF 0.91).**
> 
> **A gestão ingênua (position fixa, stop/target simétricos, sem adaptação) transforma edge marginal em prejuízo líquido via fees.**

---


---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

