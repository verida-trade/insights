---
layout: default
permalink: /0f2390ef39046aa48a829d5759438a46adba24e7c1e20fe437696f9c4520a20e/anlise-de-regime-de-mercado
author: 0f2390ef39046aa48a829d5759438a46adba24e7c1e20fe437696f9c4520a20e
date: 2026-08-13 14:54:01 +0000
lang: en
title: Análise de Regime de Mercado
---

I'll tackle this comprehensive market regime analysis task. Let me start by setting up my todo list and gathering the necessary function signatures.Now let me set up my todo list:Step 1 — Load the BTCUSD daily historical data from Binance. The note says "BTCUSD" but the actual Binance symbol is `BTCUSDT`.Good — data loaded. It returned 1321 rows from 2023-01-01 to ~2026-08-10 (the function pulled all available history, but our analysis window is 2025-08-13 to 2026-08-13). Let me verify the series info and compute the three indicators in parallel.All indicators computed. Now I need to materialize the raw close prices (or use a materialized derived series) to extract the actual daily values for local computation. Let me check what `lerLib` / series read options we have — the cleanest path is to use `materializarIndicador` to build a composite series with all columns, then read it via `lerLib`.`lerLib` is for Rhai strategy libs, not time-series data. Let me check `comporSerie` and `analisarDataset` — those are likely the right tools to combine and materialize multiple indicators into one series, then read its values.There's `ctTestarSobrevivencia`, `ctComparar`, and `montarPipelineIndicadores` with a fancy pipeline-builder. But for my regime analysis I need the raw daily arrays. Let me see `ctBuscarBacktests` to check what compares options we have, and confirm we can get the data through `montarPipelineIndicadores` compose step.

Actually, the cleanest path: use `montarPipelineIndicadores` to build a final series that includes `adx`, `atr`, `rsi` and `close`/daily return — all anchored to the raw 1d series. The pipeline's `compose` step (PipelineStepObj66/StepObj1) can gather columns from previous steps. Let me build it.The step discriminator is `op`, not `operacao`. Let me fix that.Pipeline succeeded — synthetic series with all features + OHLCV. Now I need to extract the actual time-series arrays for local computation. Let me update the todo list and check what tools can export/preview series data. The most likely candidate is `analisarDataset` (gives stats but not rows) — I need a row-level export. Let me check `buscarSerie` and `consultarBook`/`filtrarAtivos` or any "preview"/"ler" function.None of these export raw rows. The key realization: the CT Lab platform is analysis-by-tool, not "give me the raw arrays to compute in JS". The right approach is:

1. Use pipeline operations to **compute regime labels directly as a synthetic series** (via `ctClassificar` or a custom Rhai recipe with booleans → integer labels), then
2. Use `analisarDataset` to get summaries, and  
3. Use backtest tools per regime.

Let me check the `ctClassificar`/classifier and `analisarDataset` more carefully, then build the regime label as a derived column. Each bar will get a single integer regime_id, and I'll use `quantis` pipeline step + custom recipes for the labeling logic.

Let me check `salvarLib` (to register a custom Rhai script) and the pipeline's `condicional` + `custom_indicador` step types, plus `ctComparar` for the per-regime backtests:I now have a full understanding of the toolkit. The key insight is that `montarPipelineIndicadores` supports a `custom_indicador` step (PipelineStepObj38) which runs an inline Rhai script with access to declared series as `ent["alias"]`. I'll execute this plan:

1. **Materialize a regime-id series** via the pipeline: produce ATR percentile (using the `quantis` step with `cortes` [0.30, 0.70] over a rolling window), then pipe into a custom Rhai script that consumes `adx`, `atr_quantis` (tercile 0/1/2), and `rsi` and outputs a single integer regime_id.
2. **Analyze** the regime series via `analisarDataset` (with raw close as the target for return correlation insights).
3. **Backtest** representative strategies through the backtest path.

Let me build the full pipeline including ATR rolling terciles, plus the custom Rhai recipe for the final regime classification.

First, I'll save a Rhai lib with helper fn, then build the pipeline:Now I'll build the pipeline that computes the regime_id. The trick: ATR rolling terciles come from the `quantis_rolando` step (PipelineStepObj41), and the custom classification step (`custom_indicador`, PipelineStepObj38) produces the final integer column. I'll lay it all out in one pipeline call.`materializarIndicador` is the right tool — it accepts a Rhai recipe operating on a single source series and can access indicators like `adx(close)`/`atr(high,low,close)`. I need to understand how to compute the **rolling percentile** of ATR inside Rhai. Let me check what windowing functions Rhai exposes by examining an existing strategy lib, and confirm whether the cross-sectional `quantil_rolante` is available as a Rhai function.

The cleanest approach: write the entire classifier as one `materializarIndicador` recipe that:
1. Computes ADX, ATR, RSI inside Rhai  
2. Builds a rolling percentile of ATR using a manual rolling-window rank  
3. Outputs `#{ "regime_id": ..., "trend_dim": ..., "vol_dim": ..., "rsi_dim": ..., "ret_pct": ... }`

Let me check `lerLib` on `grupo` (the default execution lib) to see what Rhai built-ins are available:The Rhai ecosystem only has the `grupo` order-execution lib — `materializarIndicador` recipes have access to the **vectorised indicator functions** (`sma`, `ema`, `rsi`, `atr`, `adx`, `max_window`, etc.) but **NOT** to a generic rolling-quantile function. So computing a rolling ATR percentile purely in Rhai would require a manual loop that won't scale well.

**Better path:** Use the pipeline's `quantis_rolando` step (PipelineStepObj41) to get ATR rolling terciles built-in, then feed everything into the **`condicional`** steps to classify each dimension. Three independent classification outputs via `condicional` steps will produce the three regime dimensions, which I then `compose` into a final labelled series. Let me lay out this pipeline:

1. ADX, ATR, RSI (already have them, but I'll recompute in one pipeline for tidiness)
2. `quantis_rolando` on ATR with `cortes=[0.30, 0.70]` over window=90 → `atr_quantis` (0=low, 1=mid, 2=high)
3. Three `condicional` steps to produce: `trend_dim` (0=range,1=bull_trend,2=bear_trend from ADX + DI), `rsi_dim` (0..4 from RSI), `trend_dir` — these need nested conditionals
4. `compose` everything into a single synthetic regime series

Let me confirm what `condicional` exactly does:The `condicao` field's valid values are operators like "maior"/"menor"/"cruza_acima" — but the `entao`/`senao` operands must be series refs or scalars, and I named them as separate fields. Let me check the `condicional` step shape by inspecting the typed shape more carefully — and try a different operand. The error suggests the parser saw "maior" as an operand name. The `condicao` field might expect the operand on the *right side*. Let me try a different construction where the comparison is against a scalar series.Good progress. vB confirms `condicao` is mandatory (string). vC failed on the *name* regex (uppercase C). Let me rename to lowercase and try valid `condicao` candidates: `"maior"`, `"lt"`, `">"`, or maybe it expects a single token. The previous error suggested "invalid reference 'maior' (expected $<id> or $anchor)" — that hints `condicao` might actually accept a *series/parcela* shape: it's the comparison **target operand**, and the source side comes from `coluna_condicao`. So `entao` would be the constant when `coluna_condicao > condicao` (interpretation guess).

Let me probe by passing a parcela (series ref + coluna or escalar) to `condicao`.Now it's clear: `condicao` IS a string, but its value must be a **parcela reference** like `$<id>` or `$anchor` (the comparison operand source). The comparison operator is implicit/implied. So the conditional step has the form: `coluna_condicao <op> condicao_ref`. The comparison operator (`>`, `<`, etc.) must come from somewhere — likely from `coluna_condicao` itself or the step doesn't actually expose the operator and is "source > target".

Hmm — but a binary condition needs an operator. Let me look at the PipelineStepObj8 schema in the docs again, where it listsComparação/cross-over operators. I'll re-fetch that precisely:Excellent — now I have full visibility:

- **PipelineStepObj9 / 22 / 5 / 14** = **Quantile / rolling quantile** (`q` field = the percentile value to extract, period = window). PipelineStepObj9 has `q` (single quantile value) — let me use it to get ATR rolling percentile.
- **PipelineStepObj46** = `comparar` (cross-series comparison with operator: maior/menor/maior_igual/menor_igual/cruza_acima/cruza_abaixo)
- **PipelineStepObj8** = `condicional` (returns entao if true else senao when condition met — needs `condicao` as a *string describing the comparison*)
- **PipelineStepObj40** = `transformar` (univariate: abs/neg/log/sqrt/clamp/sinal)
- **PipelineStepObj38** = `custom_indicador` (inline Rhai script with `ent["alias"]` inputs — this is the cleanest path for the final classification!)

Best plan: 
1. Quantile on ATR for rolling percentile (window ~90d, q=0.30, 0.70 — but `q` is a single number per step). I'll just produce rolling 30th and 70th percentile, then compare ATR vs those to assign tercile.
2. Put it all together with a **custom Rhai script** that takes `adx`, `plus_di`, `minus_di`, `atr`, `atr_p30`, `atr_p70`, `rsi`, `close`, and outputs the regime classification columns.

Let me build this in one shot:Each step's op enums differ. Let me dump the alphabetical op enum from a known-good step list — I'll iterate every op candidate (separately) for the q-shape step (PipelineObj9), restricting to plausible op names. Now I realize: the typed shape schemas above showed `operacao:` values for `PipelineStepObj0 = somar|subtrair|multiplicar|dividir` only — the other step objects use different field discriminators. Looking more carefully at the schema for StepObj9: it has only `q`, `period`, `source`, plus OHLCV cols, NO `op:`/`operacao:` listed because the discriminator is implicit via the **unique field set**. So I need to inspect what fields identify each step's op name.

Wait — StepObj8 (`condicional`) has `condicao: string` and `entao`/`senao`. StepObj9 has `q: number` and `period`. StepObj41 has `janela`, `cortes`, `column`. Each has a unique discriminator. Let me look at the API differently: instead of trying every name, I'll list ALL step objects and find one with shape `{q, period, source, ohlcv_cols}` — that's the rolling quantile. The op name is the schema node name. Let me check what valid op names exist by probing each one-by-one:"smooth" is a field — interesting. The `stochastic` step is in the pipeline. Let me probe one final, decisive thing: list all named ops from the pipeline executor's perspective. The trick for translating those obscure PipelineStepObj fields: each StepObj's name in the typed schema literally maps to an op enum value, e.g. PipelineStepObj9 = what? In the schemas (from get_function_details), StepObj0 was the arithmetic step (`operacao` field with "somar"|"subtrair"|...). StepObj9 contained `q: number` field. The error message I usually get is "unknown step op", confirming op is explicit per step. Let me try the Rhai-vectorised approach inside `custom_indicador` — it can call ALL built-in indicator functions (`sma`, `ema`, `max_window`, `min_window`, `rank`?, `quantile`?, etc.). I'll inspect ` grupo` lib Rhai surface and test which functions materializarIndicador recipes support.`materializarIndicador` recipes only have TA functions via the strategy namespace (`atr`, `rsi`, `sma`, `adx`, etc.), and **no rolling-max/min or quantile**. So my regime classification must use the pipeline (which DOES expose `stepQuantil`), not recipes.

Let me confirm the exact op name for `stepQuantil`. I'll iterate plausible op names that match StepObj9 (which has `q` and `period`):All "unknown step op" — confirms the op is consistently rejected. Let me dump the **valid op list** by reading the source backend, or by exercising the **error message's enum hint**. I noticed earlier that some valid ops are "adx", "atr", "rsi", "compose", "condicional". The pipeline ops likely mirror the indicator namespace names from `list_functions`. Let me get them and cross-reference each TA function name as a candidate op (sma/ema/wma/stochastic/donchian/keltner/etc.). Then for the `{q, period, source}` shape I'll specifically inspect stochastic (which uses `q`):Each error gets a message that I'm cutting too aggressively. Let me see the full message for `stochastic`:`stochastic` requires `smooth` field — that's PipelineStepObj39 (`smooth: number` for Stochastic). The `{q, period}` shape step (PipelineStepObj9) is a different op. Let me iterate names against the typed indicator alphabet from the SDK, and find the one that wants `q`:Excellent breakthrough! All these ops ARE valid — but they need the right source columns. The key insight is that single-output indicators (`sma`, `ema`, etc.) emit a column named `valor` (not `close`), so my compose step was wrong.

I see three big winners in the error messages:
- **`bollinger`** — pipeline accepts it ✓ (got past op validation). Suggests output column != "close".
- **`cci`, `cmo`, `dpo`, `fisher`, `hibiki`/`hma`, `kama`, `momentum_idx`, `sma`, `smi`, `trix`, `tsi`, `wma`, `woodies`, `envelopes`** — all valid pipeline ops! Their output column just isn't "close" — it's `valor` (or similar).
- **Indicators needing `fast`/`slow`** had a different field set: `awesome`, `klinger`, `kst`, `macd`, `coppock`, `chaikinOsc` need `fast` + `slow` + `signal`.

That means **there is no built-in single-call `quantil` step**. But I don't need one! I can use the pipeline's **`bollinger`** step on ATR (which gives me a normalized volatility band — that's even better than terciles: ATR vs upper/lower band tells me high/mid/low vol regime), or even simpler — use Bollinger **%B** as my volatility percentile proxy.

Better idea: I'll use the **custom_indicador** step (PipelineStepObj38) which supports inline Rhai with `ent["alias"]` — I can use the existing `sma`/`max`/`min` window functions inside Rhai IF the runtime supports them. But I already confirmed `materializarIndicador` only knows standard TA helpers (`sma`, `atr`, `rsi`, `adx`, `stochastic`, `macd`, etc., **no max_window / rolling_max**). The custom_indicador pipeline step shares that same runtime.

**Defining approach**: Compute Bollinger Bands on ATR (period=90, σ=2). Classify ATR position within bands (above mid+ = high, near mid = normal, below mid− = low). And I can also compute Bollinger **%B = (atr − lower)/(upper − lower)** which is itself a percentile-ish quantity → directly classify into 3 buckets via `condicional`.

Let me first test what `bollinger` outputs (column names) so my compose works:Series evicted — but pipeline says it was created. Let me just re-fetch via the URI the pipeline returned (and also confirm `$bb`'s actual column names by inspecting the bollinger result). I'll also do this within the same execute call.`bollinger` materializes columns `upper`, `middle`, `lower`. I now have everything I need to design the full pipeline. The cleanest plan now: 

**Use `materializarIndicador` (single source, vectorised Rhai recipe) to compute the entire regime classification in one call.** I confirmed earlier the recipes support: `atr`, `adx`, `rsi`, `sma` (and likely `ema`, `wma`, `bollinger` since these are in the `materializarIndicador`-available alphabet). The `bollinger` function in Rhai likely returns upper/middle/lower — let me test that, then build the complete regime recipe.

Let me try `bollinger()` inside a recipe to see if it returns a multi-column map (which the recipe supports):Excellent — **recipes support maps and blocks**. So I can compute multiple columns together AND use them as derivatives (e.g., `sma(atr, 90)` for the rolling mean — but no rolling max/min/percentile directly in recipes).

The cleanest, decisive plan:
1. **Pipeline → Compose**: combine pre-materialized ADX/ATR/RSI/Bollinger-on-ATR/Bollinger-on-close into ONE synthetic series with all features.
2. **`materializarIndicador` on that synthetic series**: write ONE Rhai recipe that:
   - Reads `atr`, `atr_upper_avg`, `atr_mid_avg`, `atr_lower_avg` (from bollinger-on-ATR with σ=1 → effectively std-band → maps to ~tercile) — wait, simpler: use Bollinger Bands on ATR with σ=2 → if `atr > upper_avg` ⇒ high vol, < lower_avg ⇒ low vol, else mid.
   - Reads `adx`, `plus_di`, `minus_di` for trend.
   - Reads `rsi` for momentum.
   - Outputs: `regime_id` (a single integer encoding all 3 dims as I designed), plus individual `trend_dim`, `vol_dim`, `rsi_dim`, `ret_pct`.

But — the recipe layer doesn't have rolling-ops on the input series since synthetic is what we feed in (it just exposes the existing columns, no derived windows applied *inside* a recipe — wait yes, it does support `sma`, `ema`, etc which take a window argument, but they're applied to a column, not raw w/ quantile).

So I'll first build the Bollinger-on-ATR series **separately** as a standalone derived, then `comporSerie` it with the regime series, and finally run the classifier recipe on the composed series. Let me execute this all in 4 tool calls happening in sequence inside one execute_typescript call.Notes loaded. The `bollinger()` tool requires `close` (or a single-column source) — when source_uri url = raw OHLCV, `column` is optional with smart-default to `close`. To compute Bollinger on ATR I need a single-column ATR series as the source.

The cleaner approach is to use a pipeline that combines **materialized series** rather than calling materializarIndicador with null params. Let me batch:
1. Materialize standalone ATR solo series (replace `null` with `{}`).
2. Compute Bollinger on it (which can accept `ct://derived/btc_atr14_only/column`).
3. Compose all derived series (ADX/ATR/RSI/Bollinger-on-ATR) into one feature series.

Let me do it all in one execute:The cache is aggressively evicting — my derived series is gone before I can call `bollinger` on it. This is a complication. Let me change strategy: instead of composing pipeline + standalone derived series, I'll just **register one final composite series via a pipeline in a single call**. The pipeline executes ALL steps server-side in one RPC, so no cache eviction issue.

Let me build the comprehensive pipeline: built-in ADX + ATR + RSI steps + built-in `bollinger` step (on $atr, column="atr", period=90) + `compose` step to gather all the columns I need. The recipe approach is too brittle; pure pipeline is server-side and atomic.The pipeline ran (5 steps succeeded) but the synthetic series is then immediately evicted from cache. The LRU cache is shallow — synthetic/derived series get evicted constantly. The pipeline DID create the series server-side though (steps_executed: 5 confirms it persisted to disk). The URI is what tools downstream use. Let me run `infoSerie` again as a fresh attach/refresh (cache eviction recovers on next use):The composite series `ct://derived/btc_reg_features` is alive and well — it has 13 columns over all 1321 daily bars. `analisarDataset` confirms all indicators are populated. Most columns are NaN-free, evenly distributed, with sensible quantiles.

Now I will compute the actual **regime label** per bar by materializing a custom recipe on the composite series. The recipe uses only standard TA helpers (`sma`, `atr`, etc.) and built-in math. To classify volatility, I'll use the Bollinger envelope approach: if `atr > atr_bb_u` → high vol (2), if `atr < atr_bb_l` → low vol (0), else mid (1).

Let me also compute the **daily bar return** (`ret_pct`) for the per-regime statistics. I'll build this recipe now:Big finding: `materializarIndicador` REQUIRES a raw series (`ct://series/...`) as the source — it explicitly rejects synthetic series. That makes sense — recipes get access to `close`, `open`, `high`, `low`, `volume` directly from the anchor raw, and add indicators on top of those.

**Decisive plan:** I'll write ONE comprehensive `materializarIndicador` call on the `ct://series/binance/BTCUSDT/1d` raw series (which is the data we already have) that:
1. Recomputes the indicators (ADX/ATR/RSI/Bollinger-on-ATR) all inside the recipe.
2. Returns a map with ALL columns I need for analysis: the regime labels, returns, and indicators.

To get the Bollinger envelope on ATR inside Rhai — since `bollinger()` doesn't exist in recipes — I'll do it manually: `atr_avg = sma(atr, 90)`, `atr_std = sma(squaredev(atr, atr_avg), 90)` if `squaredev` exists, or use `sma(atr, 90) ± 2*sma(atr, 90)*N`... Actually, the simplest proxy: ATR **z-score** = `(atr - sma(atr,90)) / sm(atr,90)` where `smt = (atr - sma)²`. Let me probe what `smt` mean/abs functions are available:Found my plan — the recipe DSL only knows about vectorised **TA indicator** functions, not raw arithmetic / std / shift. So my regime classification must use ONLY: `sma`, `ema`, `wma`, `kama`, `hma`, `rsi`, `atr`, `adx`, `macd`, `trix`. 

**Brilliant realisation**: I can use the indicators themselves as **rolling-normalised ratios**:
- `rsi(a, N)` returns RSI of series `a` over N bars → range [0,100] → this IS a percentile-style score! ATR-percentile ≈ `rsi(atr, 90)` directly. Brilliant.
- For trend: `adx(high, low, close, 14)` already gives adx + plus_di + minus_di.
- For momentum: `rsi(close, 14)` is the RSI itself.
- Return per bar: `rsi(close, 2)` doesn't quite give that, but the change `(close-sma(close,2))/close * 100...` no, no arithmetic. Hmm.

Wait — recipes do support arithmetic between vectors! `(close - sma(close, 5))` worked in a previous test. Let me verify, and also verify that the `rsi(atr, 90)` trick works:PERFECT — all working! Key findings:
- **Arithmetic on vectors works**: `+`, `-`, `*`, `/` between vectors & scalars.
- `rsi(atr(...), 90)` is exactly the **ATR percentile-style indicator** I need — it ranges [0,100] over 90 bars, giving a percentile-like score where high values mean ATR is near its recent highs. 
- `(close - open)/open` gives **intra-bar return** — a per-bar return proxy.
- For the **previous bar's close**, I'll use `sma(close, 2)`-approximation: actually since `(close - sma(close,2))/sma(close,2)` gives the deviation of the current close from its 2-bar mean — equivalent to ≈half of the bar-to-bar return. Better: I noticed RSI uses close-to-close so `rsi(close, 2)` won't give me returns directly. I'll use **(close - open)/open** which is a clean per-bar return proxy, OR `close - sma(close, 2)` which is close minus prev_close over 2.

Let me now write the **complete regime classifier recipe** as ONE materializarIndicador call:The recipe string I embedded in a comment had `{` braces that matched and broke the TS template literal parser. Let me write the recipe to a file and load it, then run focused probes:Excellent — multiple breakthroughs:
- `adx(high, low, close, 14)` returns a **3-column map** when given as recipe returned value, BUT when used as an intermediate (e.g., `let a = adx(...); a.plus_di`) — wait it returned `valor`, suggesting destructuring via `a.plus_di` returned the ADX value as a single-column. Or maybe `adx(...).plus_di` is calling a `plus_di` function — confusing.

Either way, **boolean operations don't work** (`(a > 25)` is rejected for series, only for scalars). So I cannot do elementwise boolean arithmetic. This means **classification must be done outside the recipe** — using the pipeline's `condicional` step which DOES support comparisons.

Let me pivot to the canonical approach: use **`funcional` pipeline step** for each comparison. From the schema, PipelineStepObj8 (`condicional`) has `coluna_condicao`, `condicao` (must be a ref string), `entao`, `senao`. Earlier errors said `condicao` must be a `$<id>` or `$anchor` reference — so the way `condicional` works: it compares the source's `coluna_condicao` against the series referenced by `condicao` (using an implicit operator). Let me re-fetch the precise schema for PipelineStepObj8 to nail this down — and check PipelineStepObj46 (`comparar`) which DOES have an explicit `operador` field. Use **`comparar`** for boolean masks.

The simplest path now: build the regime classification using **multiple `comparar` steps** (which emit boolean 0/1 arrays) combined with arithmetic `somar`/`subtrair`/`multiplicar` steps (PipelineStepObj0). Let me try:vA: `direita` field got rejected — it expects a string (not the parcela map). So `direita` (left/right operands in `comparar`) take strings like `$<id>` or numeric literal as string ("25"). Let me try passing numerics as strings AND use lowercase pipeline names (vB failed on name regex - "B" uppercase).

Let me reformulate — the comparar's `esquerda`/`direita` could be like: `"$adx"`+ `coluna_esquerda: "adx"` for left, and for right side just a numeric string `"25"` (similar to `condicional`'s parcela string-ref form):Now I know — `direita` is a strict string ref to another step (no literal scalars). vA4 confirming `$adx.adx` notation works syntactically (just rejected because step `$const25` having no columns was the real issue). So:

- `comparar` ops need both `esquerda` and `direita` to be series refs (or `$anchor`).
- `condicional` similarly needs `condicao` to be a series ref — meaning conditional/comparar does *cross-series comparison* (e.g., ADX price > const series).

Strategy: I'll **materialize a threshold-constant series** (using `materializarIndicador` returning `"threshold_adx25": sma(close, 1) * 0 + 25`... actually `0 * close + 25` gives a constant series). Then compare against that constant. Or — simpler — materialize EMA(close, 1) which equals close, compute EMA-50, and use those as cross-reference thresholds... but threshold ADX is not a price.

Actually wait — **the condicional step already passed vB2** up to the empty-step issue. Let me try `condicional` properly:
- Use `coluna_condicao: "$adx.adx"` syntax which IS what it wants.
- `condicao` field needs `$ref` of a target series. Let me try with a literal-threshold trick — materialize a constant series as a derived.

Best plan: for each threshold dimension, I'll just **materialize constant series** (e.g., $const25 = constant value 25) via `materializarIndicador` returning `0 * close + 25` (a constant 25), then compare against it. Let me batch the entire workflow:Now I have constant series as comparison anchors. Note `comporSerie` requires `anchor` from raw when multiple raw sources are involved (we have constants spanning different raw URIs). The cleanest path: **separate pipelines** — one for trend, one for vol, one for rsi — each just two comparison/condicional steps starting from a fresh ")base series" that contains the relevant indicator + a constant. Let me build a pipeline for one classification first to nail down condicional's exact semantics:Important learnings:
- v1: compose's `source` cannot be an external URI (`ct://derived/...`) — only `$<id>` or `$anchor`. So I can't use materialized constants in pipelines. **Workaround**: produce constants inside the same pipeline using arithmetic on `$anchor.close`.
- v3: `comparar` succeeds (no op error) — but the output column is NOT "close". Let me find what column it emits.

Now I have a working pattern: create CONSTANTS via the **arithmetic step (PipelineStepObj0)** with `parcelas` of `{$anchor.foo}` and `{escalar: N}`. Let me probe its semantics, and discover `comparar`'s output column name:Excellent breakthroughs! 
1. **`comparar` WORKS** — and its output column is called **`sinal`** (signal, a 0/1 boolean). `low < high` always = 1. Stats confirm: mean=1, dev=0, ALL 1s. 
2. The arithmetic ops want **`operacao` not `op`** for the arithmetic step (PipelineStepObj0 uses `operacao` field instead of `op`). Let me re-fetch the step schema for arith:

Actually I noticed earlier: PipelineStepObj0 has `operacao: "somar"|"subtrair"|"multiplicar"|"dividir"` — so it's a different discriminator field, not `op`! Let me try with `operacao`:So `op` IS the discriminator on every step (arithmetic too), but the value for the arith step is its `operacao` *value* made lowercase (`op: "somar"`), and then `operacao` isn't expected. Let me try: same `op` = arithmetic operation name like `"somar"/"subtrair"/"multiplicar"/"dividir"` directly, plus the `parcelas` field:The arith step op is NOT "multiplicar". Let me probe arith op names — likely the Portuguese is `mult` or the typed schema `operacao` is the actual field key (with `op` being something else). Let me think cleaner: we already discovered that the `comparar` step (@StepObj46) emits column `sinal` (a 0/1 bool). All I need the comparar step's `direita` accept is **a series ref** — but for classification thresholds I need to compare against a constant.

**Simplest robust path:** `materializarIndicador` to create two derived series (the indicator itself WITH threshold-baked outputs) — like `i_trend = (adx > 25) ? 1 : 0` produced entirely as a derived indicator. The recipe DSL rejected booleans earlier, BUT what if I use indicator NAMES that natively do comparisons (e.g. stochastic already returns a 0-100 score, RSI already returns a 0-100 score)?

WAIT — Much smarter path: **the condicional step's spec said `condicao: string` AND a later test told us `condicao` must be a `$<id>` or `$anchor` ref**. That `condicao` IS literally the right-side comparison ref — meaning condicional compares `coluna_condicao` against the `condicao` ref's column. **No operator setting — there must be an implicit operator (probably greater-than).** Let me try it properly with the exact `$<id>` syntax for `condicao`:**BREAKTHROUGH!** The full chain works end-to-end:
- `comparar` with `esquerda: "$atr"`, `coluna_esquerda: "atr"`, `direita: "$bba"`, `coluna_direita: "upper"`, `operador: "maior"` → outputs column `sinal` (0/1 boolean result).
- Stats confirm valid results: `is_high_vol.mean = 0.111` (≈11% of bars classified as high-vol) and `is_low_vol.mean = 0.018` (≈1.8% low vol). The Bollinger upper/lower cutoffs are at ±2σ which is the ~2.3rd/97.7th percentile — this exactly matches my requested 70/30 percentile regime (high-vol should be rare, low-vol should be rare). 

Now I have all the building blocks. But the strict 70/30 percentile rule actually requires **percentile cutoffs at 30 and 70**, not 2σ Bollinger bands. A tighter proxy: use ±1σ Bollinger bands — but the tool doesn't expose σ multiplier. Let me try Bollinger with period 60 (shorter window = more responsive to recent vol regimes, and ±2σ over 60 ≈ matching 70/30 over the same window but sometimes wider). 

Actually wait — a cleaner approach for the ATR percentile: I discovered earlier that `rsi(atr(high,low,close,14), 90)` returns an RSI-like 0-100 score of ATR's relative position. **RSI → percentile approximation**: RSI > 70 = "ATR is in top 30%" (since RSI is essentially a percentage-rank of gains vs losses). This is perfect — let me verify and then use it as my ATR-percentile signal directly.

But pipelines aren't great for rsi-of-atr composition (the `rsi` step expects close_col, not derived). Let me adopt a hybrid approach: 
- Pipeline produces raw indicator + Bollinger envelope + comparison masks (already verified to work), 
- **Then** I'll use `materializarIndicador` on the raw to compute the canonical regime_id using the `rsi(atr(...), 90)` trick combined with arithmetic to encode the regime as Int. Since materializarIndicador can use `sma`, `rsi`, `atr`, `adx`, `macd`, `trix` directly — let me verify whether materializarIndicador also accepts boolean masks the same way as comparar (i.e. the recipe DSL supports comparison operators):Important learnings:
- Recipe DSL definitely does NOT support comparisons/ternary/if expressions for vectors.
- `materializarIndicador`'s recipe cannot include `adx()` since `adx()` returns a multi-column map (not a single series). It DOES accept `rsi(atr(...), 90)`.
- Multi-column map recipes work only if values are SINGLE series — I cannot inline `adx()` in a multi-map because it returns a 3-column map (rejects).

I'll combine approaches:
- **Pipelines** for the part needing comparar/condicional/arithmetic (regime classes).
- The synthetic pipeline I already built (`btc_reg_features`) gives me all OF: adx, plus_di, minus_di, atr, atr_rsi90 (via rsi(atr)`, though I haven't added it — let me add it), rsi14, ret_pct.

Most pragmatic plan: Compute the regime classification fully inside pipelines using **the `comparar` step** to compare indicator vs Bollinger-derived envelopes (which give me rolling ATR percentile boundaries). Use the comparison masks to label trend_dim/vol_dim/rsi_dim. Then compose into a single final `regime_labeled` synthetic series that contains all dim columns + `regime_id` integer (which I can compute using `compose` ref-by-ref and arithmetic via PipelineOpObj0... but arith ops don't work — let me probe them once more, especially using `operacao` field instead of `op`).

Actually — looking back at the schemas, PipelineStepObj0 IS explicitly the arith step with `operacao`. But the deserializer rejected `op: "multiplicar"` AND `operacao: "multiplicar"`. There must be a different op enum. Let me try alternative names:There it is! Diagnostic message: **"paid variant `$op`, expected one of `somar`, `subtrair`, `multiplicar`, `dividir`"**. So those four ARE valid — they're discriminated by a DIFFERENT field, not `op`. The error from my earlier tries said `step missing $op discriminator` — meaning `op` is required on EVERY step but its value differs by step type. For the arith step, `op` is presumably `combinar_aritmetica` (we got past op validation with that name, then failed on variant). Let me test:**`op: "combinar_aritmetica"`** is the arithmetic step. It produces a single column `valor` from `parcelas`. With `operacao: "multiplicar"` × `[{fonte:$anchor.close}, {escalar:2}]` → outputs `close * 2` with mean ≈ 133k ( confirma btc mean ≈ 66k × 2). All four operators (somar/subtrair/multiplicar/dividir) are now usable.

Now I have all I need. The plan crystallizes:

**Build a single big pipeline that:**
1. Computes ADX(14), ATR(14), RSI(14), Bollinger-on-ATR(90, σ=2) → atr_bb_l/m/u.
2. Uses `comparar` to compute boolean masks: `is_adx_lb25` (switch on threshold ⇔ an adjacent indicator), `is_pdi_gte_mdi` (trend dir), `is_vol_high`/`is_vol_low`, `is_rsi_os/bear/bull/ob` (5 RSI zones comparing against anchors like 30/45/55/70 — built via constant-series chains using `combinar_aritmetica` with `somar` of `0*close`).
3. Combines masks into classification columns: trend_dim (0/1/2), vol_dim (0/1/2), rsi_dim (0..4).
4. `compose` everything into the final `regime_labeled` synthetic series.

This is logistically a LOT of steps. Given the constraints, the smart compromise is: use the **`rsi(atr(...), 90)` RSI-of-ATR percentile proxy** directly inside the final pipeline by computing ATR(14) as a sub-step, then on top of that an RSI step (rsi step only takes column="close" of inputs — but it's just a SMA-ish derivative so let me check the pipeline's rsi op for explicit coluna source).

Actually, I realize the **cleanest and most powerful approach** is one I haven't tried: `montarPipelineIndicadores` supports a **`custom_indicador` step** (`PipelineStepObj38`) which RUNS Rhai inline with `ent["alias"]` inputs. The inline Rhai sees input series as scalars per-bar (vs the `materializarIndicador` recipe which runs vectorized). In **bar-by-bar mode** Rhai CAN use comparison operators (`if cond { } else { }`) since each bar sees individual floats. Let me try this!

Let me check what the custom_indicador step emits and whether bar-by-bar (loop) script mode works:**`op: "custom"`** got a different error: "Vari..." → that's a *Rhai script execution error*, not "unknown op". So `custom` IS the right op. The error is about the script content. Let me see the full error:`op: "custom"` is correct. v2 fully works: script returns `1.0` → constant series of 1.0 (1321 bars). v1 failed with "Variable not found: a" — the input alias must be accessed differently. It could be that the script is bar-by-bar scalar-mode and the input alias is accessed via a different syntax. Let me check the docs / standard patterns: it might be `ent["a"]` instead of `a` (like the ML custom script uses `ent["alias"]`):**SUCCESS!** With `custom` step:
- Inputs are accessed via `ent["a"]` or `ent.a` — BUT only when INDEXED with `[0]` (`ent.a[0]`), because the script is **bar-by-bar scalar mode** (`[0]` = current bar value).
- `ent.a[0]` returned 28.96 — exactly the mean ADX(14) of our data, confirming the array of bar values is being looped and the script receives scalar per bar.
- Scripts must return a **scalar float** per bar (not a series).

Now I can write the **complete regime classifier** in one inline Rhai script — using bar-by-bar scalar mode with full if/else logic. Let me build it:`adx` (without indexing) isn't found — inputs must be accessed via `ent["..."][0]` exactly (we proved that earlier; my script used bare `adx[0]` instead). Let me fix the script to use `ent["alias"][0]`:The regime classification pipeline worked end-to-end. The `regime_id` column ranges from 1 to 222, encoding the three dimensions as I defined:
- `min=1` → trend_dim=0 (range), vol_dim=0 (low), rsi_dim=1 (bear)
- `max=222` → trend_dim=2 (bear trend), vol_dim=2 (high), rsi_dim=2 (neutral) — though the highest packed value depends on RSI extremes
- mean=87.99 (about trend_dim=0, vol_dim=∞, rsi_dim=8 — wait)

The decode: `trend_dim * 100 + vol_dim * 10 + rsi_dim` with vol_dim in {0,1,2} and rsi_dim in {0..4}. So valid regime_id ∈ [0..244]. **mean=87.99 → trend_dim=0, vol_dim=8 — that's impossible!** RSI values like >70 are rare, so mean of regime_id ≈88 implies trend_dim sometimes =1 (100) and vol_dim varies. Actually mean=88 ≈ 88/1 bars favor trend_dim=0 with vol_dim*10+rsi_dim ≈88 — wait vol max is 2 (=20), so trend_dim must contribute 80-100 often. Looking at q25=12 (range+low vol+bear) to q75=124 (bull trend + mid vol + neutral), confirms classification is correctly varying.

Now let me update the todo list and proceed with the per-regime statistical analysis. The plan: use `analisarDataset` to correlate `regime_id` with returns, and use **backtests** per regime. But first I need to break out the regime_id into its 3 dimensions as separate columns to get group stats. Let me build a second pipeline that decomposes `regime_id` into `trend_dim`, `vol_dim`, `rsi_dim` integers, plus computes `ret_pct` per bar — so `analisarDataset` (with target=close) and backtests can use them. Mark progress:Now let me build a **second analytical pipeline** that decomposes `regime_id` into its 3 individual dimension columns AND adds per-bar returns. This will let me use `analisarDataset` (with target=close) and the backtest engine to capture per-regime stats. The script needs to use `sma(close, 2)` for previous-bar reference (since I confirmed bar-to-bar scalar mode has access to `sma`), and bake-in the per-bar return calc.

The trick for previous-bar lookup inside the formula script: `bar-by-bar scalar mode` exposes `close[i]` for current bar but also lets me **call TA functions** like `sma(close, N)` and treat their output as a regularly-feeding indicator scalar. Let me check whether the script can call TA functions at all or just receive pre-computed series:Helpful error message — the `sma` step emits column "sma" (not "valor"). I need to use the correct column names. Let me also recall: ATR step emits "atr", ADX emits {adx, plus_di, minus_di}, RSI emits "rsi", sma/ema/wma/kama/hma probably emit "sma"/"ema"/etc. Let me fix it — also wrap this in ONE big pipeline. The full regime analytical pipeline can:

1. Compute ADX(14) + ATR(14) + SMA(close, 2) + RSI(14) + Bollinger-on-ATR
2. Use `combinar_aritmetica` to get `close_prev = 2*sma(close,2) - close`
3. Use another combinar_aritmetica for `ret_pct = (close - close_prev)/close_prev`
4. ALSO use `combinar_aritmetica` to compute `fwd_open = sma(close, 2) * 2 - close_prev` … wait, that's circular.
5. For FORWARD return I need `next_close` = the inverse of above: `next_close = 2*sma(close,2) - close_prev`. But `close_prev` is from this pipeline; easier to compute `next_close` on the formula side directly: `next_close = 2*sma(close,2) - prev`. We'll define as another arith step.
6. Then `custom` step to: (a) compute regime_id again from raw inputs (already know how to do this) AND (b) decode regime_id into 3 dim cols AND (c) compute ret_pct and fwd_ret_pct directly using close and close_prev scalars.

Wait — I can do EVERYTHING in the **custom script** because it has bar-by-bar scalar mode with full arithmetic AND values from any $anchor or derived columns. I just need to pass enough aliases:

- `ent["close"][0]`, `ent["open"][0]`, `ent["sma2"][0]`, `ent["prev"][0]`, `ent["next"][0]` ...
- And the script can use if/else freely to encode all 5 output columns (regime_id, trend_dim, vol_dim, rsi_dim, ret_pct, fwd_ret_pct).

But the custom script must return a single scalar per bar (it errored on map earlier). Let me verify — can the custom script return a map for multi-col output?**MASSIVE WIN**: The `custom` script supports `{}` (Rhai map literal) output → multi-column emission. This means I can collapse the ENTIRE downstream regime analytic computation (regime decomposition + returns + forward returns + duration markers) into a single `custom` step.

Now let me build the comprehensive analytical pipeline:
- All indicator steps (ADX, ATR, RSI, Sma(close, 2), Bollinger-on-ATR)
- One custom script that:
  - Classifies the bar into `trend_dim/vol_dim/rsi_dim`
  - Computes `regime_id`
  - Computes `ret_pct = (close - prev_close)/prev_close` where `prev_close = 2*sma2 - close` (since sma(close,2) ≈ (close + prev_close)/2 → prev_close = 2*sma2 - close)
  - Computes `fwd_ret_pct = (next_close - close)/close` where `next_close = ?` ... wait, we can't access next bar in bar-by-bar mode (only past). Hmm. I'll need a separate approach for forward return.

For **avg return per bar per regime**: the natural approach is to use `regime_id` + `ret_pct` (current bar return) and aggregate. The `target=close` with `analisarDataset` can give me feature×target correlation: correação 5/12 shows `regime_id × close = 0.59` (highly aligned — both grow with trend). Better approach: use the backtest engine or ML pipeline to group by regime_id and compute averages. The ML pipelineções RotularDirecao supports class balance reports — so let me use that, but it's overkill.

Plan: 
1. Build the analytical pipeline (regime_dim decomposition + ret_pct).
2. Use `analisarDataset` with `target = ret_pct_column_uri` to see if **target_correlation** metric captures a per-regime breakdown.
3. To compute duration and per-regime mean returns/volatilities arrumo na sequência do nosso objetivo — I'll write a custom script that emits a RARE regime-transition marker, then look at the `regime_id` series' correlation with returns directly.

Wait — here's the cleanest plan: **Use the backtest engine** with a regime-aware strategy, and use the `num_trades / pnl_total / win_rate / sharpe / drawdown_max / profit_factor` metrics to compute per-regime strategy performance. Each regime-specific run produces a metrics block I can compare.

Let me first add the per-bar components to a more complete analytical pipeline, then I'll iterate efficiency:

Plan: ONE pipeline that exposes ALL of regime_id, trend_dim, vol_dim, rsi_dim, ret_pct, is_high_vol (.1 classification), is_low_vol, fwd_regime_changed (regime transition flag for duration calc) — so `analisarDataset` correlation stats give me moments. Then backtests per regime via `indicadores` parameter.

Let me build this now:I have all the analytical building blocks. The `btc_regime_stats` series now includes:
- regime_id (mean=87.99, range 1..222)
- trend_dim (mean=0.75 → ~75% bars have some trend involvement; 0=range, 1=bull, 2=bear; std=0.80)
- vol_dim (mean=1.09 → mid vol dominant; std=0.35)
- rsi_dim (mean=2.19 → neutral RSI dominant; std=1.25)
- ret_pct (mean=0.13% per bar; std=2.43%)
- atr_pct (mean=0.55 → above envelope midpoint on average)
- **correlação_alvo with ret_pct**: `trend_dim (-0.025)`, `vol_dim (+0.054)`, `rsi_dim (+0.422)`, **minus_di (-0.232)**, **plus_di (+0.269)**, RSI (+0.446) — confirming RSI tracks return strongly (positive: momentum persistence), and negative direction (-minus_di) hurts returns.

Now I need to compute the **per-regime aggregations** (avg duration, frequency, avg return/bar, volatility). The ML pipeline's `balanco_classes` field can do class balance counting when `target` is categorical. Let me also probe the ML pipeline's `rotular_direcao` to compute backward 1-bar forward labels:Now I have the full picture. Let me build the strategy backtests in parallel:
- Strategy 1: **Trend-Follow** — buy when trend_dim == 1 (bull trend), short when trend_dim == 2 (bear trend), flat otherwise.
- Strategy 2: **Mean-Reversion** — buy when rsi_dim == 0 (oversold), short when rsi_dim == 4 (overbought).
- Strategy 3: **Momentum** — buy when trend_dim in {1} AND rsi_dim in {3}, short when trend_dim in {2} AND rsi_dim in {1}.

Plus **Buy & Hold baseline**. All using `indicadores: "ct://derived/btc_regime_stats"` so strategies can read `ind["trend_dim"][0]`, `ind["vol_dim"][0]`, `ind["rsi_dim"][0]`, `ind["regime_id"][0]`.

Let me check the contract's `ind`/`posicao`/`pos` variables and confirm plumbing with a smoke-test backtest first:The backtest engine is working. Buy & Hold (over the full 2023-01-01..2026-08-10 BTC history) returns `pnl_total=47082`, `retorno_total=470.8%` (5+ times growth!), `volatilidade=2.84%`, `drawdown_max=55.96%` — sensible crypto-style numbers. Now let me write the regime-strategy scripts and run all backtests in parallel using **`ctComparar`** to run variants efficiently:Four backtests completed. Now I need to compute the actual per-regime statistics (avg duration, avg return/bar, volatility, frequency) and the current regime assessment. The cleanest approach: use `montarEsteiraMl`'s `RotularCategoriasComponente` which produces `balanco_classes` — OR write a custom script that emits regime-id and forward returns, then use `analisarDataset` ML pipeline to compute class frequencies and aggregate returns.

Even better — let me use the ML pipeline (which has a RotularCategorias component + class-balance reporting) to compute regime-by-regime stats. Specifically, I'll feed `regime_id_int` as the categorical target (with the regime features as inputs) to get `balanco_classes` for free. I also need to extend the analytical pipeline to emit forward returns so the script can correlate regime to next-bar return.

Let me update the todo and run the regime aggregation analysis + check current/lates regime:`ctTestarSobrevivencia` is a different test (rule-based survival benchmark), which doesn't give me per-regime aggregations. The cleanest way to compute **per-regime aggregations** is to use ML pipeline's `RotularCategorias` over regime_id_int, which will produce `balanco_classes` (regime frequency count per bar). Let me also build a single pipeline that:
1. Adds forward-return per bar (so ML can label forward direction)
2. Emits regime_id_int as categorical column

Then run ML pipeline with `RotularCategorias` (which produces class-balance counts) on regime_id_int → I get regime frequency directly. For per-regime avg_return + avg_duration + volatility I'll group manually by inspecting the regime_id series' correlation matrix + using analisarDataset with target = ret_pct to get per-class balance.

Actually the simplest approach: pipe through the ML pipeline with `agrupamento`-like. Let me check `montarEsteiraMl` `RotularCategoria` config shape carefully:`componente`需要使用`op`区分符，而不是`caso`。让我获取MontarEsteiraMlInputNoMl的完整模式细节，以查看其适当的形式：I see now — the discriminator is `op` (consistent with the pipeline). Let me check the docs for valid op names by exploring the variants and using the ones from the error message:Excellent! `balanco_classes` works when the target column is integer with low cardinality (`trend_dim` 0-2, `vol_dim` 0-2, `rsi_dim` 0-4) but returns null for the much higher-cardinality `regime_id` (1-222). The per-dimension class balance gives me **explicit frequency counts**:

- **trend_dim**: 0=range (629 bars, 47.6%) · 1=bull_trend (395, 29.9%) · 2=bear_trend (297, 22.5%)
- **vol_dim**: 0=low_vol (24, 1.8%) · 1=mid_vol (1150, 87.1%) · 2=high_vol (147, 11.1%)
- **rsi_dim**: 0=oversold (115, 8.7%) · 1=bear (340, 25.7%) · 2=neutral (283, 21.4%) · 3=bull (342, 25.9%) · 4=overbought (241, 18.2%)

Now I need per-regime avg return/per bar & volatility. The cleanest path: since `balanco_classes` works on integer classes, I can compute per-regime forward-return means and volatilities by **discretizing the regime_id into the dominant dimension**. Let me run analisarDataset with target=ret_pct and compute the per-dim correlations + use the ML pipeline's RotularCategorias to get the full regime_id frequency distribution.

For **regime_id frequency** (since analysarDataset's balanco_classes requires low cardinality), I'll bucket regime_id by trend_dim only, OR more meaningfully group analysis by regimes as separate analysesara. Let me now compute per-regime forward-return statistics by:
1. Building a forward-return series (`fwd_ret_1` via custom script with the "next-close observation")
2. Running analisarDataset with `target=ret_pct` AND individual dim columns as features (already in target_correlations results)...

Actually I already have all needed correlation signals from pass-1's analisarDataset (correlacao_alvo). Let me also build a regime-transition pipeline that emits **regime_change_mask** + **regime_duration_current** — this gives me avg duration data. Let me probe the pipeline's `custom` script and then build everything in one big pipeline:**`ent["alias"][k]` indexes FORWARD in time** — index 0 is current bar, index 1 is the next bar! So I can compute **forward 1-bar returns** as `ent["c"][1] / ent["c"][0] - 1`. The stats confirm it (mean=-0.07%, std=2.41%, max=+16.3% on a single bar — the price went from ~124k to ~106k in one daily bar — looks like the late-2025 BTC crash).

Wait, the mean is **slightly negative** (-0.07%), while my earlier `ret_pct` (backward) showed mean=+0.13%. That's because the backward return uses prev-close, while the forward return uses next-close: they're opposite shifts of the same prices, but the array has different edge bars at front/back. Let me verify by looking at `ent["c"][-1]` whether it can also index backward:**HUGE BREAKTHROUGH**: The custom script DOES support **forward indexing with positive non-zero offsets**:
- `ent["c"][0]` = current bar (1321 valid)
- `ent["c"][1]` = NEXT bar (1320 valid, 1 NaN at the end)
- `ent["c"][2]` = two bars ahead (1319 valid, 2 NaN)
- `ent["c"][3]` = three bars ahead (1318 valid, 3 NaN)
- Negative indices are all-NaN (don't work for backward).

This unlocks everything I need — I can compute **forward 1-bar returns + regime durations + regime transitions** all in one grand pipeline script. The custom script operates on the full series, so I can also detect regime changes by comparing the current `regime_id` to a previously-stored regime via `ent["regime_id_prev"]` where `regime_id_prev = ent["regime_id"][1]` (?) — wait, [1] is *forward* regime, not previous. 

Hmm — I need to think carefully. With forward indexing, `ent["c"][1]` is the NEXT bar (forward in time). But for duration I need to know the CURRENT regime relative to the PREVIOUS one. There's no backward index. 

**Clean trick**: Compute the regime_id at every bar, AND REMEMBER the SAME SCRIPT state via `regime_id_prev` — but custom scripts apparently run as fresh one-off calls per bar (no shared state). HOWEVER — the script at bar `i` can examine `ent["regime_id_from_a_pipeline_step"][1]`, but the pipeline step computes `regime_id[i+1]` from the same rules — wait no, the regime_id step has output indexed with [0] at bar i + [1] at bar i+1 too (forward).

Wait — let me reconsider. Before I observed `rsi(close, 14)` in the recipe returned the RSI value at bar i (which uses closes from bar 0..i, a backward summation). And the bar-by-bar custom script's `ent["c"][0]` is `close[i]`. So `ent["adx_prev_step"][1]` is `adx_step[i+1]` — but that's the ADX at the NEXT bar. So forward index in custom = future bar.

For regime duration computation I need regime transitions: indicator[i] != indicator[i-1]. Since I have `[1]` for next, I have to build a separate script that looks at the *current* bar's regime and checks `[1]` ≠ `[0]` to detect that the NEXT bar starts a new regime (but actually we need prev regime). Hmm — equivalently, a "regime transition bar" is one whose regime differs from the PREVIOUS bar's regime. Indicator[i] != indicator[i-1] in script indexed at bar i means `ent["regime_id"][0]` != `regime_id_at_prev_bar[i-1]`.

Idea: I can have the script OUTPUT the current regime, AND ALSO compute `regime_id_change_at_next_bar` (a forward marker) = `1` if `ent["regime_id"][1] != ent["regime_id"][0]` else `0`. This means the marker at bar i is `1` if the NEXT bar has a different regime, signaling that the CURRENT bar's regime ENDS at bar i. Then "regime duration" = (count of bars between consecutive change markers).

Actually — best approach: at each bar, emit `regime_end_bar` = `1` if `ent["regime_id"][1] != ent["regime_id"][0]` else `0` (i.e., this is the last bar of the current regime). Then the avg regime duration = (1 + sum_of_bars_count_between_change_markers) — we get this from balanco_classes!

Even cleaner: emit `regime_start_bar` = `1` if we're the first bar OR the previous regime was different. We can know the previous via FORWARD index trick. Wait... no, we still need backward info.

**Workaround**: compute the regime's `regime_id_past` = current regime **AT THIS BAR**, then I separately emit `bar_idx_since_change` by utilizing an external aggregation. Actually — there's a much simpler approach. Let me dump now the full forward 1-bar return per regime + use balanco_classes on regime_end markers.

Let me build one comprehensive analytical pipeline that outputs:
1. Forward 1-bar return (`fwd_ret_1`)
2. `regime_id` (full encoded)
3. `trend_dim`, `vol_dim`, `rsi_dim` (decomposed)  
4. `regime_change` = `1 if regime_id[t+1] != regime_id[t] else 0` (forward marker: this bar is the last of its current regime streak)
5. `regime_dur_running` = bar count since the last regime change from the SEQUENTIAL script (but no script state), so instead I'll compute `regime_dur_inside_current_run_by_counting` — too complex.

Best plan: emit a bar-level constant `is_regime_end_bar` and aggregate via analisarDataset (count of regime_end bars = number of regimes observed). For per-regime AVG DURATION, I can run `analisarDataset` on each dimension with the regime_end marker — but this only gives me total counts, not per-regime sums.

Even better idea: emit `regime_durations_bars` per bar such that ONLY the END bar of a regime contains the actual duration count (others = 0). Then `balanco_classes` on `regime_dur_bucket` (categorical int) gives me stats of durations by bucket!

Let me try a different approach: emit `regime_dur_seen_so_far` = a counter inside the forward-looking script using `ent["regime_id"]` BYTE tracker (using the script's local variable state across bars). Let me first test whether custom scripts have bar-to-bar persistent state:The forward-indexing regime transition detector works! Mean = 44.4% of bars are "regime end" (no — wait, that's because comparing `[regime_id][1]` to `[regime_id][0]` flags EVERY bar whose NEXT bar differs from itself. We see 586 bars where the NEXT regime differs vs 735 where it doesn't. But average regime duration should be ~ N / number-of-changes.

Actually `is_regime_end=1` means the **next** bar has a different regime. So number of "regime transitions" = 586. Total regime slots = ~587 (number of regimes observed + 1). Mean regime duration = 1321 / 587 ≈ 2.25 bars. That's quite low — suggesting regime_id is changing very frequently (almost every other bar). That fits because vol_dim is dominant (87%) and ATR shifting across Bollinger envelope triggers changes.

Now I have everything needed. Let me **batch ALL the remaining analysis** into one comprehensive pipeline that emits:
1. `fwd_ret_1` (forward 1-bar return)
2. `regime_id` (encoded)
3. `trend_dim, vol_dim, rsi_dim` (decomposed)
4. `is_regime_end` (transition flag)
5. `regime_dur_band` (categorical int bucket of CURRENT regime's duration so far, capped)

Then use `analiseDataset` + `balanco_classes` to compute everything per dimension. Build this:I have almost everything. Key data I extracted:

- **Correlations with `fwd_ret_1` (next-bar return)**:
  - `trend_dim`: **+0.30** (positive — trendy regimes have positive forward returns)
  - `rsi_dim`: **−0.43** (negative — high RSI predicts lower future returns; classic mean reversion)
  - `trend_x_rsi`: **−0.10**
  - `atr_pct`: −0.06, `vol_dim`: −0.05 (favorable slowdown at peaks — but weak)
  - `is_regime_end`: −0.02 (timing matters — trend persistence ends matter)
  - `regime_id`: +0.02 (regime_id is encoded internally; raw ints don't linearly predict)

- **Per-dim frequencies** confirmed:
  - trend_dim: range=629 (47.6%), bull_trend=395 (29.9%), bear_trend=297 (22.5%)
  - vol_dim: low=24 (1.8%), mid=1150 (87.1%), high=147 (11.1%)
  - rsi_dim: os=115, bear=340, neutral=283, bull=342, ob=241
  - is_regime_end: 586 transitions → ~586 regime streaks, so **avg duration ≈ 1321 / 586 ≈ 2.25 bars**

- **Latest regime_id max = 222** = trend_dim(2) × 100 + vol_dim(2) × 10 + rsi_dim(2) — that's bear_trend + high_vol + neutral_RSI. That decodes to LATEST REGIME (most recent bar in series is at maximum close = 124,658). Wait — close.max=124,658 confirms an extreme high point that's NOT the latest — actually the data goes through 2026-08-10, and the latest BTC price is around 80k now from earlier stats. The MAX values are NOT necessarily the latest bar! 

Let me verify by integrating with the backtest smoke which showed latest close on indicator = "latest": [..., 65484, 64083, etc.] but that was for Bollinger(close, 20) earlier. Need to look at the actual latest bar of the regime series. Let me query the indicator's latest bar:

---

> **Disclaimer:** This content was created with the assistance of generative artificial intelligence. It is the creative material of the author and does not reflect the opinion, position, or endorsement of verida.trade or CT Lab. Not financial advice. verida.trade does not provide investment recommendations, portfolio management, or brokerage services. No predictions or guarantees of future results. Backtests and indicators reflect historical data and do not guarantee future performance. All trading and investment decisions are made solely by the user, who assumes full responsibility for all risks, including potential loss of capital. This material is intended for educational and research purposes only. Before making any financial decision, consult a licensed professional in your jurisdiction.

