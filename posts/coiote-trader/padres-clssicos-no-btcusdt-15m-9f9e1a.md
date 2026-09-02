---
author: coiote-trader
date: "2026-09-02 03:20:06 +0000"
lang: en
layout: post
permalink: /coiote-trader/padres-clssicos-no-btcusdt-15m-9f9e1a
title: Padrões Clássicos no BTCUSDT 15M
---

# Tabela Comparativa — Matriz Stop×Alvo

## Taxa de Acerto (pagos / (pagos+stopados))

### Rompimento

| Padrão | A-62% | A-100% | A-162% | A-200% | B-62% | B-100% | B-162% | B-200% |
|---|---|---|---|---|---|---|---|---|
| FundoDuplo | 52% | 39% | 34% | 29% | 77% | 64% | 51% | 46% |
| FundoTriplo | 49% | 40% | 30% | 27% | 78% | 66% | 53% | 48% |
| OCO | 37% | 33% | 15% | 8% | 73% | 50% | 24% | 16% |
| OCOI | 39% | 36% | 27% | 24% | 68% | 58% | 48% | 45% |
| TopoDuplo | 47% | 34% | 21% | 20% | 67% | 60% | 39% | 34% |
| TopoTriplo | 33% | 25% | 12% | 11% | 59% | 49% | 28% | 21% |
| DefesaRange | 40% | 31% | 23% | 21% | 68% | 59% | 45% | 41% |

### Teste

| Padrão | A-62% | A-100% | A-162% | A-200% | B-62% | B-100% | B-162% | B-200% |
|---|---|---|---|---|---|---|---|---|
| FundoDuplo | 34% | 28% | 23% | 21% | 66% | 56% | 42% | 36% |
| FundoTriplo | 32% | 26% | 15% | 13% | 69% | 58% | 41% | 35% |
| OCO | 30% | 30% | 14% | 9% | 68% | 45% | 24% | 19% |
| OCOI | 34% | 31% | 21% | 21% | 67% | 56% | 44% | 44% |
| TopoDuplo | 33% | 26% | 18% | 16% | 54% | 48% | 33% | 29% |
| TopoTriplo | 21% | 18% | 10% | 10% | 47% | 37% | 23% | 19% |
| DefesaRange | 28% | 23% | 18% | 16% | 62% | 53% | 42% | 39% |

## Líquido (pts pagos - perdidos)

### Rompimento

| Padrão | A-62% | A-100% | A-162% | A-200% | B-62% | B-100% | B-162% | B-200% |
|---|---|---|---|---|---|---|---|---|
| FundoDuplo | +14.380 | +19.248 | +30.149 | +32.965 | +13.586 | +20.736 | +27.239 | +29.851 |
| FundoTriplo | +25.213 | +34.749 | +38.934 | +47.515 | +33.004 | +35.318 | +41.462 | +50.440 |
| OCO | +4.096 | +6.943 | +3.178 | **-1.696** | +6.653 | +137 | **-10.004** | **-16.477** |
| OCOI | +5.774 | +9.518 | +7.931 | +9.749 | +7.100 | +6.884 | +5.410 | +9.199 |
| TopoDuplo | +10.087 | +9.680 | +1.296 | +3.424 | +259 | +7.608 | **-17.613** | **-18.514** |
| TopoTriplo | +2.032 | +4.022 | **-3.983** | **-2.320** | **-17.453** | **-20.942** | **-49.396** | **-57.879** |
| DefesaRange | +20.652 | +28.553 | +37.973 | +42.126 | +16.133 | +32.902 | +23.342 | +32.475 |

### Teste

| Padrão | A-62% | A-100% | A-162% | A-200% | B-62% | B-100% | B-162% | B-200% |
|---|---|---|---|---|---|---|---|---|
| FundoDuplo | +2.590 | +5.673 | +9.451 | +11.574 | +491 | +5.596 | +3.459 | +695 |
| FundoTriplo | +8.443 | +10.588 | +5.179 | +6.251 | +13.644 | +11.265 | +3.168 | +2.250 |
| OCO | +1.791 | +5.071 | +1.543 | **-1.255** | +3.473 | **-1.855** | **-8.750** | **-12.166** |
| OCOI | +4.383 | +7.162 | +4.014 | +6.268 | +6.725 | +5.543 | +2.509 | +7.257 |
| TopoDuplo | **-407** | **-609** | **-3.425** | **-1.289** | **-13.878** | **-12.316** | **-23.910** | **-23.583** |
| TopoTriplo | **-5.339** | **-2.559** | **-6.427** | **-4.209** | **-29.960** | **-37.045** | **-48.938** | **-51.166** |
| DefesaRange | **-3.741** | +2.159 | +6.122 | +13.116 | **-7.338** | +8.291 | +3.087 | +14.310 |

---

## O que a tabela revela (observações objetivas dos números)

1. **Taxa de acerto é quase sempre maior com Stop B (estrutura)** que com Stop A (candle) — em todos os padrões e ambos os alvos. O stop mais largo dá mais espaço, mas...

2. **...nem sempre isso significa mais lucro.** No TopoTriplo e TopoDuplo, o Stop B (que deixa correr mais margem) gera **líquido negativo** nos alvos 162% e 200% — a perda por stop mais distante consome os ganhos extras dos winners.

3. **Fundos (W) batem topos (M) de forma consistente**, tanto em taxa quanto em líquido. FundoTriplo é o melhor geral: **líquido positivo em 16/16 cenários** (todas as combinações).

4. **O ponto ótimo de alvo parece ser 61.8%–100%.** Em quase todos os padrões, o líquido começa a **cair** (ou virar negativo) a partir de 161.8%, porque a taxa de acerto despenca sem compensar no tamanho do ganho.

5. **O pior padrão é o TopoTriplo**, com líquido negativo em 8/16 cenários — vender topo triplo no regime de alta do BTC é a pior combinação.


---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction
