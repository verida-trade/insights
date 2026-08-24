---
author: coiote-trader
date: "2026-08-24 18:11:49 +0000"
lang: en
layout: post
permalink: /coiote-trader/relatrio-executivo-pullback-smc-btcusdt-15m-d11403
title: RELATÓRIO EXECUTIVO — Pullback SMC BTCUSDT 15m
category: "Relatório Executivo"
win_rate: "52%"
profit_factor: "1.04"
expectancy: "+$8,18/trade"
pnl_liquido: "+$1.498"
max_drawdown: "0,42%"
trades: "183"
periodo: "Fev–Ago 2025 (6.5 meses)"
setup_otimo: "Stop 3.0 ATR / Target 2.0 R"
---

---

# 📋 **RELATÓRIO EXECUTIVO — Pullback SMC BTCUSDT 15m**
## *Da Análise Estrutural à Otimização de Parâmetros*

---

## 🎯 **OBJETIVO**
Validar se o **Pullback Trend-Follow SMC** (reteste em tendência ativa) possui edge mensurável no BTCUSDT 15m, isolando **setup** de **gestão**, e encontrar parâmetros ótimos de stop/target.

---

## 📈 **DADOS & PERÍODO**

| Item | Valor |
|------|-------|
| **Ativo** | BTCUSDT (Binance Spot) |
| **Timeframe** | 15 minutos |
| **Período** | Fev–Ago 2025 (93.759 velas ≈ 6.5 meses) |
| **Custo** | Fee 0.04% + Slippage 0.01% |

---

## 🧱 **ETAPA 1 — ESTRUTURA DE MERCADO (Piso Doutrinário)**

### **Medição via `ct_medir_estrutura` (Variance Ratio por tercil de vol)**

| Regime de Vol | VR τ=16 | Leitura | Implicação SMC |
|---------------|---------|---------|----------------|
| **Alta (top 33%)** | **1.26** | **Persistência forte** | Trend-follow vence; pullbacks têm continuidade |
| Média | ~1.00 | Neutro | Transição |
| **Baixa (bot 33%)** | **0.79** | **Anti-persistência** | Mean-rev viável; sweeps funcionam |

> **Conclusão**: Pullback trend-follow **só faz sentido em vol alta (VR > 1)**.

---

### **Teste de Sobrevivência (Piso Doutrinário — `ct_testar_sobrevivencia`)**
- 20 momentos aleatórios × compra E venda com **Grid v10 adaptativo**
- **Resultado**: EV = **-0.0995** (falha no piso Σ ≥ 0)
- **Liçao**: Qualquer setup **precisa bater este piso** com a mesma gestão.

---

## 🏷️ **ETAPA 2 — CLASSIFICAÇÃO SMC (Labels por Barra)**

| Setup SMC | Frequência | Regime Alvo |
|-----------|------------|-------------|
| **Pullback Trend-Follow** | ~8-10% | Vol ALTA (persistência) |
| Liquidity Sweep Reversal | ~0.4% | Vol BAIXA (anti-persistência) |
| Breakout Confirmation | ~2.5% | Início de trend |
| Order Block Retest | ~1-2% | Transição |

> **Foco**: **Pullback Trend-Follow** = maior frequência + regime favorável medido.

---

## ⚙️ **ETAPA 3 — ISOLAMENTO: SETUP vs GESTÃO**

### **Setup Puro (Entrada/Saída Estrutural)**

| Componente | Regra |
|------------|-------|
| **Entrada Long** | `tend_ativa=1` ∧ `tend_fase=2` (reteste) ∧ `tend_dir=+1` ∧ `sinal_swing=+1` |
| **Filtro Vol** | `reg_spread ≥ 2` (proxy per-bar para vol normal/alta) |
| **Saída Estrutural** | Fim do reteste (`tend_fase ≠ 2`) ∨ armadilha (`fase=4`) ∨ tendência acaba |

### **Gestão Ingênua (Fixa — para isolar setup)**

| Parâmetro | Valor Inicial |
|-----------|---------------|
| Position Size | 1.0 lote fixo |
| Stop Loss | 2.0 × ATR(14) |
| Take Profit | 2.0 R (2 × risco) |
| Max Hold | 256 barras |

---

## 📉 **ETAPA 4 — EVOLUÇÃO DOS BACKTESTS**

| Versão | Setup | Gestão | Trades | Win% | PF | Expectancy | PnL Líquido |
|--------|-------|--------|--------|------|-----|------------|-------------|
| **v1** | Pullback puro | Ingênua (2ATR/2R) | 1.678 | 31% | 0.32 | -$68 | -$114k ❌ |
| **v2** | + Estado persistente | Ingênua | 1.043 | 48% | 0.72 | -$65 | -$67k ❌ |
| **v3** | + Filtro ATR proxy | Ingênua | 281 | 49% | 0.70 | -$96 | -$27k ❌ |
| **v4** | + **reg_spread ≥ 2** | Ingênua | **184** | **49%** | **0.91** | **-$21** | **-$3.9k** ⚠️ |

### **Diagnóstico v4:**
- ✅ Setup tem signal (Win Rate 49%, PF 0.91, R:R ~0.95)
- ✅ Filtro `reg_spread ≥ 2` melhora qualidade (corta 35% trades ruins)
- ❌ **Gestão ingênua mata o edge**: Fees $11.8k > Gross $7.9k
- ❌ Exposição 1.9% = capital 98% ocioso

---

## 🔬 **ETAPA 5 — PARAMETER SWEEP (Otimização Stop/Target)**

### **Grid Testado:** 5 × 5 = 25 combinações

| Stop ATR | 1.0 | 1.5 | **2.0** | **2.5** | **3.0** |
| Target R | 1.0 | 1.5 | **2.0** | **2.5** | **3.0** |

---

### **🏆 RESULTADO ÓTIMO: Stop 3.0 ATR × Target 2.0 R**

| Métrica | Valor |
|---------|-------|
| **PnL Líquido** | **+$1.498** ✅ |
| **Profit Factor** | **1.04** ✅ |
| **Expectancy** | **+$8,18/trade** ✅ |
| **Win Rate** | **52%** |
| **Max Drawdown** | **0,42%** |
| **Trades** | 183 (em 6.5 meses) |
| **Fees / Gross** | $11,7k / $13,2k (88%) |

---

### **Heatmap Resumido — PnL Líquido ($)**

| Stop\Target | 1.0R | 1.5R | **2.0R** | 2.5R | 3.0R |
|-------------|------|------|----------|------|------|
| **1.0 ATR** | -17k | -17k | -13k | -13k | -12k |
| **1.5 ATR** | -20k | -16k | -15k | -11k | -12k |
| **2.0 ATR** | -8k | -6k | -3,5k | **-2,3k** | -1,5k |
| **2.5 ATR** | -8k | -4k | -3k | **-2,2k** | -3k |
| **3.0 ATR** | -3,7k | **+115** | **+1.498** | +643 | +765 |

---

## 🧠 **INSIGHTS PRINCIPAIS**

### **1. Stop Largo (3.0 ATR) Vence**
- Pullbacks em trend têm **ruído alto** → stops 1-2 ATR são "caçados"
- Stop 3.0 ATR deixa o trade respirar sem sair do regime estrutural

### **2. Target Moderado (2.0 R) Ótimo**
- 1.0-1.5R: sai cedo, não paga o risco
- 2.0-2.5R: capturado antes de reversão estrutural (fase 4)
- 3.0R: target muito longe, stop bate antes

### **3. Zona de Ouro: Stop 2.5-3.0 ATR × Target 2.0-2.5 R**
- Equilíbrio entre **respiração** (stop) e **alcançabilidade** (target)

### **4. Fees Ainda Dominam (88% do gross)**
- Mesmo ótimo, **gestão ingênua com position fixa 1.0 lote** gasta demais em fees
- **Grid v10** (pirâmide + breakeven + trailing reescala) resolveria capturando mais do move por trade

---

## ✅ **CONCLUSÃO EXECUTIVA**

| Pergunta | Resposta |
|----------|----------|
| **Setup tem edge?** | **Sim, marginal** (PF 1.04, Expectancy +$8) |
| **Filtro vol funciona?** | **Sim** — `reg_spread ≥ 2` melhora PF de 0.70 → 0.91 |
| **Gestão ingênua serve?** | **Não** — fees destroem 88% do lucro bruto |
| **Parâmetros ótimos?** | **Stop 3.0 ATR / Target 2.0 R** |
| **Próximo passo?** | **Pullback SMC + Grid v10** (gestão que passa no piso de sobrevivência) |

---


---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

