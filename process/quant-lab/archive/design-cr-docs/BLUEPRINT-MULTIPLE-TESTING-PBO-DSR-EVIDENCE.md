---
status: draft-for-cp3
version: "0.1"
cr_id: "CR-164"
source_story_map: "docs/product/STORY-MAP.md"
source_mvp_scope: "docs/product/MVP-SCOPE.md"
confirmed_by: ""
confirmed_at: ""
---

# Blueprint: CR164 Computable Multiple-Testing / PBO / DSR Evidence

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-12 | host-orchestrator inline meta-se-critical | 冻结四方法能力边界、唯一数据 owner、既有 consumer 集成、五个 CP4 outcome 输入。 |

## 能力地图

| Capability | 用户价值 | 产品候选 | Owner Feature |
|---|---|---|---|
| CAP-164-01 Evidence input binding | 证明所有方法消费同一 sealed family | S01 | FEAT-164-01 Contract & validator |
| CAP-164-02 Multiple-testing correction | 产生 BH 与 WRC/SPA 可追溯结果 | S02 | FEAT-164-02 Multiple-testing calculators |
| CAP-164-03 Overfit/deflation evidence | 产生 PBO/CSCV 与 raw-count DSR 结果 | S03 | FEAT-164-03 Overfit calculators |
| CAP-164-04 Conservative projection | 防止单方法 PASS 覆盖失败或不可用 | S04 | FEAT-164-04 Aggregation & consumers |
| CAP-164-05 Independent verification | 证明确定性、fail-closed、权限与回归 | S05 | FEAT-164-05 Verification fixtures |

## Feature / Epic 边界

| Feature | 负责事项 | 不负责事项 | 拥有数据 | 只读数据 | 禁止依赖 |
|---|---|---|---|---|---|
| FEAT-164-01 | input envelope、schema、identity/sufficiency validation、availability | 统计计算、admission policy | normalized input envelope、validation result | CR163 sealed lineage、candidate statistics/return matrix | consumer 反向写 input；后验缩小 family |
| FEAT-164-02 | BH、stationary-bootstrap WRC/SPA、parameter provenance | PBO/DSR、consumer 最终判定 | BH/WRC-SPA method evidence | FEAT-01 validated input | 直接写 admission package |
| FEAT-164-03 | CSCV split、PBO、raw-count DSR 与 count non-alias | effective-trial estimator | PBO/DSR method evidence | FEAT-01 validated input | 将 raw count 写为 effective count |
| FEAT-164-04 | severity lattice、reason codes、existing consumer projections | 重新计算方法、创建平行 gate | aggregate summary、consumer projection | FEAT-02/03 evidence | 修改 CR163 lineage；改善更差状态 |
| FEAT-164-05 | fixture/static tests、10-run hash、CR155/permission regression | 真实研究批次、生产数据 | test fixtures/evidence index | 所有 contract 与 outputs | runtime/data/provider/broker access |

## 跨 Feature 流程

| Flow | 触发 | 参与 Feature | 写入 Owner | 失败路径 | 验证入口 |
|---|---|---|---|---|---|
| FLOW-164-01 | sealed family + method inputs | 01 → 02/03 → 04 | 每个阶段只写自己的 typed artifact | missing→typed_unavailable；conflict/tamper/invalid→blocked | SCN-CR164-P01/N01/F01 |
| FLOW-164-02 | consumer attach | 04 → CR151/CR154/package | FEAT-04 写 projection；consumer 只读 | worst-state merge；不得 OR-pass | disagreement fixtures |
| FLOW-164-03 | fixture verification | 05 → all | FEAT-05 只写 test evidence | 任一 false PASS/overclaim 阻断 CP7 | QAC-001..010 |

## 共享能力与集成契约

| Shared | Owner | 调用方向 / 时机 | 输入 | 输出 | 降级 / 调用方修改 |
|---|---|---|---|---|---|
| Evidence input validator | FEAT-01 | calculators 调用前 | sealed lineage + complete candidate membership + method payloads | validated envelope / typed status | 缺失 unavailable；冲突 blocked；producer 需提供 explicit refs/hash/count |
| Method evidence envelope | FEAT-01 | calculator 完成后 | method id/config/input/output/provenance | immutable JSON-safe typed evidence | 非有限数 blocked；FEAT-02/03 统一封装 |
| Conservative aggregator | FEAT-04 | 四方法 evidence 齐集后 | mandatory method evidence | aggregate status/reasons/refs | `BLOCKED > FAIL > TYPED_UNAVAILABLE > PASS`；consumer 只接 projection |

## 五个 CP4 outcome 输入

| Candidate | Outcome | Dependency | 建议 lld_policy |
|---|---|---|---|
| CR164-S01 | contract/input validator | 无 | full-lld |
| CR164-S02 | BH + WRC/SPA evidence | S01 | full-lld |
| CR164-S03 | PBO/CSCV + raw-count DSR evidence | S01 | full-lld |
| CR164-S04 | conservative aggregation + 3 consumer projections | S01-S03 | full-lld |
| CR164-S05 | QAC/negative/determinism/permission verification | S01-S04 | full-lld |

## 待确认边界

| Decision ID | 类型 | 问题 | 推荐方案 | 备选 | 优劣 / 切换条件 |
|---|---|---|---|---|---|
| DQ-CP3-CR164-001 | architecture | calculator 与 evidence contract 如何组织 | 一个 method-neutral envelope + 四个纯 calculator + 单一 aggregator | 单体 calculator；插件 registry | 推荐边界清晰且无需 runtime registry；方法数显著扩张再考虑 registry。 |
| DQ-CP3-CR164-002 | architecture | WRC/SPA block length | `fixed_window` 为 MVP 默认，显式正整数、seed/config 全量入 provenance | automatic selector | 固定值最可复跑；只有另行验证 automatic selector 后切换。 |
| DQ-CP3-CR164-003 | architecture | disagreement 最终状态 | severity lattice + claim-specific mandatory set | 任一 PASS 即 PASS；多数票 | 推荐符合 CP2 no-OR-pass；多数/OR 会过度声明。 |
| DQ-CP3-CR164-004 | security | 是否批准五 Story 输入且保持 design-only | 批准后进 CP4；实现仍等 CP5 | 修改设计；暂停 | 不扩大任何 runtime/data/外部权限。 |

