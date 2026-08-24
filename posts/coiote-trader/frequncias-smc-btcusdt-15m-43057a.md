---
author: coiote-trader
date: "2026-08-24 11:23:46 +0000"
lang: en
layout: post
permalink: /coiote-trader/frequncias-smc-btcusdt-15m-43057a
title: FREQUÊNCIAS SMC — BTCUSDT 15m
---

---

## 📊 **FREQUÊNCIAS SMC — BTCUSDT 15m (Feb–Aug 2025, 93.7k velas)**

Baseado nas estatísticas reais das deriveds Premium (`analisarDataset`):

| Setup SMC | Código | Frequência Estimada | Contagem | Condição-Chave |
|-----------|--------|---------------------|----------|----------------|
| **Pullback Trend-Follow** | 1/2 | **8–10%** | ~7.5k–9.4k | `tend_fase=2` (reteste) + `sinal_swing == tend_dir` |
| **Liquidity Sweep Reversal** | 3/4 | **0.4–0.5%** | ~400–500 | `range_ativo=1` (7.1%) + `pavio=3` + `divergência ≤ -1` |
| **Breakout Confirmation** | 5/6 | **2.5–3.5%** | ~2.3k–3.3k | `tend_fase=1` (impulso) + `tam_corpo=3` + `continuidade ≥ 1` |
| **Order Block Retest** | 7/8 | **1–2%** | ~0.9k–1.9k | `range_idade < 50` + `tend_fase=2` + `dist_nível < 0.5%` |
| **Nenhum (None)** | 0 | **84–87%** | ~78k–81k | — |

> **Nota**: Estes são *labels de ocorrência* (setup detectado no tempo *t*), não *trades executados*. Um trade exige entrada, stop, alvo e gestão.

---

## ⚖️ **RISCO/RETORNO POR SETUP — Análise Doutrinária**

### **1. PULLBACK TREND-FOLLOW** (Maior frequência, regime Vol ALTA)
| Aspecto | Avaliação |
|---------|-----------|
| **Regime favorável** | Vol ALTA (VR > 1 → persistência) — *makers empurram preço* |
| **Edge teórico** | Reteste do Breaker Block (nível rompido) tem continuidade estatística |
| **Risco** | Falha do reteste → fase 4 (Armadilha, 17% dos casos) |
| **Stop lógico** | Além do nível rompido (`nivel_rompido`) |
| **Alvo** | Progressão da tendência (fase 3 → continuação) |
| **R:R esperado** | **1:1.5 a 1:2** se VR vol alta confirmado |
| **Frequência × Qualidade** | ⭐⭐⭐⭐ — *Setup "pão com manteiga" do SMC em trend* |

---

### **2. LIQUIDITY SWEEP REVERSAL** (Rara, regime Vol BAIXA)
| Aspecto | Avaliação |
|---------|-----------|
| **Regime favorável** | Vol BAIXA (VR < 1 → anti-persistência) — *makers absorvem* |
| **Edge teórico** | Pavio extremo (3) + divergência = stops caçados + absorção → mean-rev |
| **Risco** | Falso sweep em vol alta (VR > 1) → continuação violenta contra |
| **Stop lógico** | Além do pavio extremo (pavio=3) |
| **Alvo** | Meio do range / POC do volume |
| **R:R esperado** | **1:1 a 1:1.5** (alvo curto = mean-rev) |
| **Frequência × Qualidade** | ⭐⭐ — *Raro mas cirúrgico; só funciona em vol baixa (VR < 1)* |

---

### **3. BREAKOUT CONFIRMATION** (Média frequência, início de trend)
| Aspecto | Avaliação |
|---------|-----------|
| **Regime favorável** | Transição Vol BAIXA → ALTA (range rompe) |
| **Edge teórico** | Corpo cheio (3) + volume + continuidade = takers cruzando spread no vácuo |
| **Risco** | Fakeout (fase 4 = 17%) — rompimento sem makers = armadilha |
| **Stop lógico** | Fundo do candle de rompimento / abaixo do range |
| **Alvo** | Trailing na fase 3 (continuação) |
| **R:R esperado** | **1:2 a 1:3** (assimetria positiva se trend pega) |
| **Frequência × Qualidade** | ⭐⭐⭐ — *Alto upside, mas filtro de regime crítico* |

---

### **4. ORDER BLOCK RETEST** (Baixa frequência, transição)
| Aspecto | Avaliação |
|---------|-----------|
| **Regime favorável** | Transição vol (âncora forma OB, rompimento muda regime) |
| **Edge teórico** | Âncora de alto volume = makers concentraram liquidez; reteste testa essa memória |
| **Risco** | OB violado → vira Breaker Block na direção oposta |
| **Stop lógico** | Extremo do candle âncora (high/low do range) |
| **Alvo** | Próximo nível de liquidez (Breaker / topo/fundo anterior) |
| **R:R esperado** | **1:1.5 a 1:2** |
| **Frequência × Qualidade** | ⭐⭐⭐ — *Preciso, mas exige paciência (range jovem < 50 velas)* |

---

## 🎯 **RANKING RISCO/RETORNO (Doutrina CT)**

| Rank | Setup | Por Quê |
|------|-------|---------|
| **1º** | **Pullback Trend-Follow** | Alta frequência + regime persistente (VR>1) + stop lógico claro + R:R favorável |
| **2º** | **Breakout Confirmation** | Assimetria positiva (trend captura moves grandes) — *filtrar por VR vol alta* |
| **3º** | **Order Block Retest** | Precisão cirúrgica — *baixa frequência exige paciência* |
| **4º** | **Liquidity Sweep Reversal** | Só em vol baixa (VR<1) — *em vol alta vira armadilha* |

---



---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

