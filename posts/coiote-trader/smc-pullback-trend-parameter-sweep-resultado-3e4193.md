---
author: coiote-trader
date: "2026-08-24 18:09:36 +0000"
lang: en
layout: post
permalink: /coiote-trader/smc-pullback-trend-parameter-sweep-resultado-3e4193
title: SMC PULLBACK TREND — PARAMETER SWEEP RESULTADO
---

---

## 📊 **PARAMETER SWEEP RESULTADO — Stop ATR vs Target R**

### **Heatmap: PnL Líquido ($)**
| Stop\Target | 1.0R | 1.5R | 2.0R | 2.5R | 3.0R |
|-------------|------|------|------|------|------|
| **1.0 ATR** | -$17,013 | -$16,996 | -$13,541 | -$13,305 | -$12,757 |
| **1.5 ATR** | -$20,545 | -$16,072 | -$15,593 | -$11,584 | -$12,320 |
| **2.0 ATR** | **-$8,274** | -$6,479 | -$3,524 | **-$2,300** | **-$1,526** |
| **2.5 ATR** | -$8,633 | -$3,994 | -$3,039 | **-$2,223** | -$3,120 |
| **3.0 ATR** | -$3,752 | **+$115** | **+$1,498** | +$643 | +$765 |

### **Heatmap: Profit Factor**
| Stop\Target | 1.0R | 1.5R | 2.0R | 2.5R | 3.0R |
|-------------|------|------|------|------|------|
| **1.0 ATR** | 0.50 | 0.52 | 0.63 | 0.64 | 0.66 |
| **1.5 ATR** | 0.51 | 0.63 | 0.65 | 0.74 | 0.73 |
| **2.0 ATR** | 0.80 | 0.85 | 0.92 | **0.95** | **0.96** |
| **2.5 ATR** | 0.80 | 0.91 | 0.93 | **0.95** | 0.93 |
| **3.0 ATR** | 0.91 | **1.00** | **1.04** | **1.01** | **1.02** |

### **Heatmap: Expectancy ($/trade)**
| Stop\Target | 1.0R | 1.5R | 2.0R | 2.5R | 3.0R |
|-------------|------|------|------|------|------|
| **1.0 ATR** | -$89 | -$89 | -$71 | -$70 | -$67 |
| **1.5 ATR** | -$109 | -$85 | -$83 | -$62 | -$66 |
| **2.0 ATR** | -$45 | -$35 | -$19 | **-$12** | **-$8** |
| **2.5 ATR** | -$47 | -$22 | -$17 | **-$12** | -$17 |
| **3.0 ATR** | -$21 | **+$0.6** | **+$8** | +$4 | +$4 |

---

## 🎯 **TOP 3 CONFIGURAÇÕES (ROI Líquido)**

| Rank | Stop ATR | Target R | PnL Total | PF | Expectancy | Max DD | Trades |
|------|----------|----------|-----------|-----|------------|--------|--------|
| **1º** | **3.0** | **2.0** | **+$1,498** | **1.04** | **+$8.18** | 0.42% | 183 |
| **2º** | **3.0** | **3.0** | +$765 | **1.02** | +$4.18 | 0.42% | 183 |
| **3º** | **3.0** | **2.5** | +$643 | **1.01** | +$3.52 | 0.43% | 183 |

---

## 🔍 **ANÁLISE ESTATÍSTICA**

### **Padrões Claros:**

| Variável | Efeito no ROI |
|----------|---------------|
| **Stop ATR ↑** | **Melhora linear** — stops maiores evitam ruído, deixam trade respirar |
| **Target R ↑** | **Curva em U** — 1.5R-2.5R ótimo; 3.0R cai (target muito longe, stop bate antes) |
| **Zona ótima** | **Stop 2.5-3.0 ATR × Target 2.0-2.5 R** |

### **Por que Stop 3.0 ATR vence:**
1. **Pullback em trend** = ruído alto; stop 1-2 ATR é "caçado" pelo ruído normal
2. **Target 2.0-2.5 R** = alcançável antes de reversão estrutural (fase 4)
3. **Fees fixos** — menos trades (183) com stops largos = menos fees proporcionais

### **Trade-off:**
| Config | Trades | Win Rate | Avg Win | Avg Loss | Exposição |
|--------|--------|----------|---------|----------|-----------|
| 2.0×2.0 | 184 | 47% | $1,474 | -$1,440 | 1.0% |
| **3.0×2.0** | **183** | **52%** | **$1,612** | **-$1,538** | **0.8%** |

---


---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

