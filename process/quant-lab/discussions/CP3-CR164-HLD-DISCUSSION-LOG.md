# CP3 CR164 HLD Discussion Log

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-12 | host-orchestrator inline meta-se-critical | 基于 CP2 已批准范围完成四个 Architecture Gray Areas 的 table-first 分析。 |

## 执行模式

- 用户指令：`不要拉起子agent，继续推进quant-lab项目`。
- 执行方式：用户批准的 `inline-fallback`；未启动、未恢复、未消息任何子 Agent。
- 证据：`process/state/AGENT-DISPATCH-LEDGER.ndjson` 的 `AD-CR164-CP3-META-SE-CRITICAL-INLINE-20260712T195927+0800`。
- 模型声明：无 child model；不声称使用 `.codex/agents/meta-se-critical.toml` 指定模型。

## 方案形成结论

| Gray Area | 推荐 | 未选方案 | 影响 |
|---|---|---|---|
| calculator boundary | method-neutral envelope + four pure calculators | monolith / dynamic registry | 可独测、无 runtime discovery |
| WRC/SPA bootstrap | stationary bootstrap + explicit fixed window | automatic selector | 确定性与 provenance 优先 |
| DSR count | explicit raw-count mode；effective unavailable/non-alias | raw copied to effective | 防止 claim overreach |
| disagreement | `BLOCKED > FAIL > TYPED_UNAVAILABLE > PASS` | OR-pass / majority | 符合 CP2 mandatory conservative semantics |

## 理论来源核验

- Politis & Romano (1994) 定义 stationary bootstrap；未据此推断 automatic block selector 已被本项目实现。
- White (2000) Reality Check、Hansen (2005) SPA 与 Bailey/López de Prado DSR 作为方法族来源；精确公式/version/golden fixture 必须在后续 LLD 冻结。

## Deferred / switch conditions

- automatic block selector：明确算法、版本、reference fixtures 与独立验证后另行 ADR。
- effective trial estimator：独立产品/设计范围，不由 raw-count DSR 隐式引入。
- plugin registry / artifact service：方法生态或并发/跨包需求出现时另起 CR。
- real ML/event adapters 或真实研究批次：需独立授权，不属于 CR164 当前 CP3。

## 待 CP3 决策

`DQ-CP3-CR164-001..004` 全部进入正式 CP3 Decision Brief；无额外 blocking clarification。

