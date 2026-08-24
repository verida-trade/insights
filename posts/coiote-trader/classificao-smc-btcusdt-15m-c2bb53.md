---
author: coiote-trader
date: "2026-08-24 05:49:34 +0000"
lang: en
layout: post
permalink: /coiote-trader/classificao-smc-btcusdt-15m-c2bb53
title: CLASSIFICAÇÃO SMC — BTCUSDT 15m
---

---

## 📊 **CLASSIFICAÇÃO SMC — BTCUSDT 15m (Feb–Aug 2025, 93.7k velas)**

### 1️⃣ **PISO DOUTRINÁRIO — Estrutura do Caminho (VR por tercil de vol)**

| Regime | VR τ=4 | VR τ=16 | VR τ=64 | Leitura Doutrinária |
|--------|--------|---------|---------|---------------------|
| **Vol ALTA (tercil sup)** | **1.12** | **1.26** | **1.22** | **PERSISTÊNCIA FORTE** — makers empurram preço (trend-follow vence) |
| **Vol MÉDIA** | 0.98 | ~1.00 | 1.04 | Neutro / transição |
| **Vol BAIXA (tercil inf)** | **0.92** | **0.79** | **0.75** | **ANTI-PERSISTÊNCIA** — makers absorvem (mean-rev viável) |

> **Implicação SMC**: Em vol alta, *Order Blocks / Breakers* tendem a **falhar como reversão** — o preço respeita a direção do fluxo (BOS → continuação). Em vol baixa, *Liquidity Sweeps* nos extremos do range têm **maior probabilidade de mean-reversion**.

---

### 2️⃣ **CANDLE CLASSIFICADO — 18 Eixos SMC (últimas 100 velas = amostra)**

| Eixo | Código | Leitura SMC | Distribuição (q25/q50/q75) |
|------|--------|-------------|----------------------------|
| **Direção** | `direcao_candle` | -1=vendedor pagou bid / +1=comprador pagou ask | -1 / 0 / +1 |
| **Tamanho do Corpo** | `tamanho_corpo` 0..3 | 0=pavio só / 3=corpo cheio (convicção taker) | 1 / 2 / 3 |
| **Pavio Inferior** | `pavio_inferior` 0..3 | **Makers defendendo bid** (absorção compradora) | 1 / 2 / 2 |
| **Pavio Superior** | `pavio_superior` 0..3 | **Makers defendendo ask** (absorção vendedora) | 1 / 2 / 2 |
| **Simetria Pavios** | `simetria_pavios` 0..3 | 0=assimétrico (lado único) / 3=simétrico (indecisão) | 1 / 2 / 3 |
| **Posição Close no Range** | `posicao_close_no_range` 0..4 | 0=low / 4=high (onde takers fecharam) | 1 / 2 / 3 |
| **Regime Spread** | `regime_spread` 0..3 | 0=compressão (book fino) / 3=expansão (volatilidade) | 1 / 1 / 1 |
| **Regime Volume** | `regime_volume` 0..4 | 0=seco / 4=explosivo (urgência taker) | 1 / 1 / 1 |
| **Divergência Esforço×Resultado** | `divergencia_esforco_resultado` -2..+2 | **-2=volume alto + corpo pequeno = absorção** (Order Block candidate) | -1 / 0 / +1 |
| **Continuidade Direção** | `continuidade_direcao` 0..2 | 2=mesma direção 3+ velas (fluxo sustentado) | 1 / 1 / 2 |
| **Sobreposição Candle Anterior** | `sobreposicao_candle_anterior` 0..3 | 3=engolfo / inside bar (transição de liquidez) | 1 / 1 / 2 |

**Padrões SMC Detectáveis via Classificação:**
- **Order Block (Bullish)**: `divergencia_esforco_resultado ≤ -1` + `pavio_inferior ≥ 2` + `tamanho_corpo ≤ 1` + `direcao_candle = -1` (vendedor pagou bid, mas makers absorveram)
- **Order Block (Bearish)**: `divergencia_esforco_resultado ≤ -1` + `pavio_superior ≥ 2` + `tamanho_corpo ≤ 1` + `direcao_candle = +1`
- **Breaker Block**: `sobreposicao_candle_anterior = 3` (engolfo) + mudança de `regime_spread` + `continuidade_direcao = 2` na nova direção
- **FVG / Imbalance**: `is_outside_bar = 1` + `gap_pct_change ≠ 0` + `regime_volume ≥ 2` (vácuo de liquidez)
- **Liquidity Sweep**: `pavio_inferior = 3` ou `pavio_superior = 3` + `regime_volume ≥ 3` + `divergencia_esforco_resultado = -2` (stops caçados + makers retiram liquidez)

---

### 3️⃣ **SWINGS — BOS / CHoCH (via ct_bop calibrado Pareto 80/20)**

| Coluna | Significado SMC |
|--------|-----------------|
| `sinal_swing` | **-2=Reversão Baixa (CHoCH↓)** / **-1=Pullback Baixa** / **+1=Pullback Alta** / **+2=Reversão Alta (CHoCH↑)** |
| `lado_ct_bop` | Força compradora (+1) / vendedora (-1) / neutro (0) |
| `regiao_ct_bop` | 0=zero / 1=intermediário / 2=extremo (quantil rolante) |
| `virada_ct_bop` | Mudança de lado detectada (-1, 0, +1) |

**Estatísticas**: `sinal_swing ≠ 0` ocorre em ~**12-15% das velas** (pullbacks + reversões). `±2` (reversões puras) ~**2-3%**.

---

### 4️⃣ **RANGES — ORDER BLOCKS / BREAKER BLOCKS (ct_range)**

| Métrica | Valor | Leitura |
|---------|-------|---------|
| `range_ativo` média | **7.1%** | Apenas 7% do tempo em range formal (âncora alto vol + reversão) |
| Idade média do range | **22.6 velas** (~5.6h) | Ranges curtos — BTC 15m trend-heavy |
| Topo/Fundo médio | 77k / 76k | Níveis onde makers concentraram liquidez (Order Blocks institucionais) |

> **Regra SMC**: Range nasce em **candle âncora** (vol z-score ≥ 3 + movimento prévio ≥ 2 ATR + próxima barra reverte). Morre no **close fora de [fundo, topo]** → rompimento = BOS.

---

### 5️⃣ **TENDÊNCIA PÓS-Rompimento — FASES SMC (ct_tendencia)**

| Fase | Código | Leitura SMC | Frequência |
|------|--------|-------------|------------|
| **Inativo** | 0 | Sem tendência ativa (dentro de range) | ~7% |
| **Impulso** | 1 | **Breakout** — takers cruzam spread no vácuo (FVG) | ~23% |
| **Reteste** | 2 | Preço retorna ao `nivel_rompido` (testa Order Block / Breaker) | ~18% |
| **Continuação** | 3 | **Trend confirmed** — makers reposicionam liquidez na direção | ~35% |
| **Armadilha** | 4 | **Fakeout** — rompimento falha, inverte pro lado oposto | ~17% |

**Estado Atual (última vela)**: `tend_ativa=1`, `direcao=+1` (alta), `fase=1` (Impulso), `progresso=1.79` (1.79× altura do range acima do topo rompido), `nivel_rompido=76.731`

---

## 🎯 **MAPA SMC CONSOLIDADO — COMO LER O BTCUSDT AGORA**

```
┌─────────────────────────────────────────────────────────────────┐
│  REGIME ATUAL: Vol ALTA (tercil sup) → VR 1.12-1.35 (PERSISTÊNCIA) │
├─────────────────────────────────────────────────────────────────┤
│  ESTRUTURA DE PREÇO (SMC)                                        │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐            │
│  │  RANGE      │──▶│  BOS ↑      │──▶│  IMPULSO    │  (fase 1)  │
│  │  (OB zona)  │   │  @ 76.731   │   │  progresso  │  1.79×     │
│  └─────────────┘   └─────────────┘   └─────────────┘            │
│       │                  │                  │                    │
│       ▼                  ▼                  ▼                    │
│  Order Block       Breaker Block      Continuação /            │
│  (âncora vol)      (engolfo +        Armadilha (fase 4)         │
│  absorção)         novo equilíbrio)  17% prob.                  │
└─────────────────────────────────────────────────────────────────┘
```

### **Rótulos SMC por Contexto de Volatilidade**

| Contexto | Setup SMC | Prob. Doutrinária | Validação Necessária |
|----------|-----------|-------------------|----------------------|
| **Vol ALTA + VR>1** | Trend-follow: Pullback HL (fase 2→3) | **ALTA** | `ct_tendencia.fase=2` + `swing.sinal_swing=+1` + `candle.divergencia≥0` |
| **Vol ALTA + VR>1** | Reversão em Order Block | **BAIXA** | Makers *empurram* preço — OB vira Breaker |
| **Vol BAIXA + VR<1** | Mean-rev: Liquidity Sweep nos extremos | **MÉDIA-ALTA** | `range_ativo=1` + `candle.pavio=3` + `divergencia=-2` |
| **Vol BAIXA + VR<1** | Breakout de range (BOS) | **BAIXA** | Vácuo sem makers — armadilha provável (fase 4) |

---


---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

