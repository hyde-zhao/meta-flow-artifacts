---
status: "completed-pending-cp3-human-gate"
change: "CR-153"
checkpoint: "CP3"
created_at: "2026-07-02T17:20:00+08:00"
owner: "host-orchestrator"
discussion_type: "Architecture Gray Areas"
source_context: "process/context/CP3-CR153-EVENT-DRIVEN-STRATEGY-E2E-HLD-CONTEXT.yaml"
---

# CP3 CR153 HLD Discussion Log

## 1. Discussion Scope

CR153 CP2 已由用户批准，9 个 CP2 决策项均接受推荐方案。本 CP3 讨论只处理事件驱动策略 E2E foundation 的架构灰区，不授权 Story decomposition、LLD、源码实现、真实数据、runtime、broker、credential、catalog/event/model registry 或 external framework 操作。

## 2. Architecture Gray Areas

| AGA ID | Question | Option | Pros | Cons | Recommendation | Assumptions / When to Switch |
|---|---|---|---|---|---|---|
| AGA-CR153-01 | Event admission gate 应独立还是塞进 CR151/CR152 gate？ | A. Event-specific gate + CR151/CR152 四态 adapter | 保留事件研究方法语义，复用共享状态治理。 | 需要新增 event companion gate。 | Recommended | 若 CP5 证明事件证据完全可无损映射到现有 gate，可把 event gate 缩成 thin adapter，但不得污染 CR151/CR152 schema。 |
| AGA-CR153-02 | Event metadata contract 是否等同 event store/catalog/registry？ | A. Metadata-only contract | 满足 fixture/static 验证，不产生写入副作用。 | 不提供事件存储、发布或生产 current truth 能力。 | Recommended | 任何 store/catalog/registry write 都需独立 CR 与 runtime/storage authorization gate。 |
| AGA-CR153-03 | 是否重建 event strategy framework？ | A. Extend existing `ResearchDatasetSpec` / `BacktestRunSpec` / `StrategyAdmissionPackage` anchors | 避免平行框架，保留 admission 链路。 | 需要 contract delta table 和 adapter。 | Recommended | 若锚点被发现缺陷，优先修复或窄包装锚点，不静默 fork。 |
| AGA-CR153-04 | CP3 是否允许真实事件 feed 或 live listener 证明？ | A. Deterministic local fixtures only | 可审计、无外部副作用、匹配 CP2 边界。 | 不能证明真实 feed、实时性或生产 readiness。 | Recommended | 真实 feed/listener 需后续 runtime/data gate。 |
| AGA-CR153-05 | CV/survivorship/capacity/impact 等完整治理是否进入 CR153？ | A. First-wave slots only; full governance deferred to CR154 | 控制范围，保留机器可见 blocker/ref slot。 | CR153 不能声明完整生产策略可靠性。 | Recommended | 用户显式要求提前完整治理时，需重开 CP2 或新增 CR。 |

## 3. Advisor Table Summary

| Decision Surface | Recommended CP3 Position | Rationale | Downstream Obligation |
|---|---|---|---|
| Gate relation | Event-specific gate + shared four-state adapter | Event study evidence differs from multifactor statistical and ML evidence. | CP5 Story must test PASS / FAIL / NEEDS_REVIEW / BLOCKED mapping. |
| Metadata boundary | Event metadata only, no store/catalog/registry write | Prevents fixture contract from being mistaken for production event infrastructure. | CP5 must include forbidden operation counters and no-write checks. |
| Contract anchors | Extend existing anchors | Keeps strategy research/backtest/admission chain converged. | Story design must include anchor delta table. |
| Validation mode | Local/static/fixture-only | Matches CP2 approval and avoids real runtime/data risk. | CP7/CP8 wording must state contract semantics only. |
| CR154 split | Slots in CR153, full governance later | EV-GAP-8/9 remain visible without scope creep. | CP3 and later gates must preserve deferred risk list. |

## 4. Human Decision Items for CP3

The following CP3 decisions are ready for the CP3 Decision Brief:

| Decision ID | Recommended Option |
|---|---|
| DQ-CP3-CR153-001-EVENT-GATE-ADAPTER | Event-specific admission gate plus CR151/CR152 four-state adapter. |
| DQ-CP3-CR153-002-EVENT-METADATA-NO-STORE | Event metadata contract only; no event store/catalog/registry writes. |
| DQ-CP3-CR153-003-EXTEND-EXISTING-ANCHORS | Extend/compose existing ResearchDatasetSpec, BacktestRunSpec and StrategyAdmissionPackage anchors. |
| DQ-CP3-CR153-004-FIXTURE-ONLY-VALIDATION | Deterministic local/static fixtures only. |
| DQ-CP3-CR153-005-CR154-DEPENDENCY | Full CV/survivorship/capacity/impact/regime/reconciliation governance deferred to CR154. |

## 5. Boundary Confirmation

This discussion does not authorize implementation. CP3 approval, if granted later by the user, only allows CP4 Story planning and later CP5 design evidence preparation under the same no-runtime/no-real-data/no-store-write boundary.

