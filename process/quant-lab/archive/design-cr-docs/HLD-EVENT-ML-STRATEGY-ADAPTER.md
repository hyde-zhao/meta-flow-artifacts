---
status: "cp3-approved"
version: "0.2"
change: "CR-158"
title: "Event + ML Strategy Adapter Unified Implementation HLD"
created_at: "2026-07-05T17:20:00+08:00"
owner: "host-orchestrator"
confirmed_by: "user"
confirmed_at: "2026-07-05T17:35:00+08:00"
scope_boundary: "local/static/fixture-only adapter contracts; no runtime, no real data access, no model training, no registry write, no implementation before CP5"
companion_docs:
  - "docs/design/ARCHITECTURE-DECISION-EVENT-ML-STRATEGY-ADAPTER.md"
  - "docs/design/BLUEPRINT.md#cr158-增量event--ml-strategy-adapter-unified-implementation"
---

# HLD: Event + ML Strategy Adapter Unified Implementation

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-05 | host-orchestrator | CR158 CP3 初稿：定义 thin shared core + typed event/ML extensions、refs-only evidence、no-runtime guard、候选方案、场景模拟和 CP3 待决策项。 |
| 0.2 | 2026-07-05 | host-orchestrator | CP3 approved：确认 thin shared core + typed extensions，并保持 no-runtime/no-real-data/no-registry/no-publish 边界。 |

## 1. 问题定义

CR157 已关闭为 `READY_WITH_RISK`，并明确把 event strategy adapter 与 ML strategy adapter implementation 延后。CR158 在 CP2 已批准将 `DF-CR157-001` 与 `DF-CR157-002` 合并为一个统一 CR，但统一不等于强行把两种策略压进同一字段集。

CP3 的问题是：如何定义一个可进入 Story 拆解的架构，使 event 与 ML strategy adapter 共用最小公共合同，同时把事件语义和 ML 语义隔离在 type-specific extension 中，并继续遵守 CR157/CR158 的 refs-only evidence 和 no-runtime/no-publish 边界。

## 2. 目标、成功标准和非目标

| ID | 目标 | 可度量成功标准 |
|---|---|---|
| G1 | 固化统一 adapter 架构 | HLD 至少给出 3 个候选方案、1 个推荐方案、3 个 ADR 和 2 个 CP3 DQ。 |
| G2 | 明确 shared core 与 type-specific extension | Shared core 字段不少于 7 类；event extension 字段不少于 6 类；ML extension 字段不少于 7 类。 |
| G3 | 保持 evidence refs-only | HLD / ADR / CP3 Brief 至少 3 处明确大型正文复制计数必须为 0。 |
| G4 | 保持 no-runtime / no-publish | HLD / ADR / CP3 Brief 至少 3 处明确 forbidden operation counters 全部为 0。 |
| G5 | 支持 CP5 设计证据 | 输出可拆为 5-6 个候选 Story 组，并明确 CP5 前不得实现。 |

| 非目标 | 边界 |
|---|---|
| Story decomposition | CP3 通过后才进入 CP4；本 HLD 只给候选 Story group。 |
| LLD / implementation | CP5 通过前不得改 adapter source/test implementation。 |
| Real event feed | 不接入 live listener、provider、gateway 或真实 event stream。 |
| Real ML training | 不训练模型、不调用外部模型服务、不写 model registry。 |
| Production/runtime readiness | 不声明 simulation-ready、paper-ready、live-ready、trading-ready、production-ready 或 registry-published。 |

## 3. Architecture Gray Areas

| Gray Area | 来源 | 为什么影响 HLD | 推荐处理 |
|---|---|---|---|
| AGA-CR158-01 shared core 厚度 | REQ-CR158-001、SGQ-CR158-001 | 决定 schema owner、Story 切分和 event/ML 字段是否互相污染。 | thin shared core + typed extensions。 |
| AGA-CR158-02 evidence extension 归属 | REQ-CR158-004、SC-CR158-P02 | 决定 event/ML refs 是进入 adapter schema、evidence index 还是 handoff schema。 | Adapter owns typed refs shape；evidence index stores refs-only records。 |
| AGA-CR158-03 no-runtime enforcement 层级 | REQ-CR158-005、SC-CR158-N02 | 决定 guard 写在 adapter validator、shared safety layer、还是只在 CP7 文档审查。 | Adapter validator reports counters; FEAT-07 policy validates all counters are 0。 |
| AGA-CR158-04 HLD 拆分 | HLD 拆分原则、Story candidates CR158-S01..S06 | Event 与 ML 可拆，但共享 core / evidence / guard 强耦合。 | 保持单份 HLD；若 CP5 发现 shared core 不成立再拆子 CR。 |

### Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Thin shared core + typed extensions | 公共消费路径统一；event/ML 字段不互设必填；CP5 可按 core/event/ML/evidence/guard 分 Story。 | 需要新增 extension boundary 和 adapter validation。 | schema、HLD、ADR、Story split、tests、release wording。 | Recommended | 若 extension 仍互相污染，CP5 拆分子 CR 或改两套 adapter。 |
| B. Fat common schema | 字段集中、单 schema 易展示。 | event-only / ML-only 字段会互相成为噪声或错误必填；验证复杂。 | schema、tests、文档、后续维护。 | Not recommended | 仅当 event 与 ML 字段在 CP5 被证明高度一致时切换。 |
| C. Two independent adapters | 语义最清晰，局部实现简单。 | 重复 HLD/CP5/CP7；evidence/handoff/no-runtime 容易分叉。 | CR governance、Story planning、verification。 | Backup | 若 CP3/CP5 证明 shared core 不可用时切换。 |

## 4. 推荐架构

推荐 Option A：`StrategyTypeAdapterCore` + `EventAdapterExtension` + `MLAdapterExtension` + `AdapterTypedEvidenceRef` + `AdapterOperationCounterReport`。

```text
fixture/static event refs       fixture/static ML refs
        |                              |
        v                              v
EventAdapterExtension          MLAdapterExtension
        \                              /
         \                            /
          v                          v
             StrategyTypeAdapterCore
                      |
                      v
        AdapterValidationResult / BlockedReasons
                      |
        +-------------+-------------+
        |                           |
        v                           v
AdapterTypedEvidenceRef      AdapterOperationCounterReport
        |                           |
        v                           v
ResearchEvidenceIndex       FEAT-07 no-runtime guard
        \                           /
         \                         /
          v                       v
              Stage 2 / Stage 3 handoff refs
```

### Shared Core 字段

| 字段类 | 说明 | 必填性 |
|---|---|---|
| `adapter_id` / `strategy_type` | 标识 adapter 与类型：event / ml。 | required |
| `input_refs` | 指向 type-specific fixture/static refs。 | required |
| `output_signal_refs` | 指向统一 signal refs，不复制 signal body。 | required |
| `evidence_refs` | 指向 typed evidence refs。 | required |
| `blocked_reason_refs` | 缺失、不可用、未授权或验证失败的结构化 reason。 | required when blocked |
| `authorization_flags` | no-runtime/no-publish/no-registry/no-feed 等边界。 | required |
| `handoff_refs` | Stage 2 / Stage 3 消费所需 refs。 | required |

### Type-specific Extensions

| Extension | 字段类 | 失败路径 |
|---|---|---|
| Event | event source ref、event time ref、payload schema ref、alignment policy ref、signal output ref、blocked reason ref | 缺 source/alignment/signal 或 feed/listener/provider/gateway counter 非 0 时 blocked。 |
| ML | training snapshot ref、feature set ref、label policy ref、model artifact ref、validation report ref、prediction signal ref、blocked reason ref | 缺 training snapshot/validation/prediction 或 training/service/registry counter 非 0 时 blocked。 |

## 5. 模块职责

| 模块 / 区域 | 职责 | 输入 | 输出 | 失败路径 |
|---|---|---|---|---|
| Adapter core contracts | 定义 shared core、typed extension envelope、validation status 和 blocked reasons。 | fixture/static event refs、fixture/static ML refs。 | adapter core object、extension object、validation result。 | 字段缺失、type mismatch、extension 污染 core -> blocked。 |
| Event adapter extension | 表达事件输入、时间对齐、payload schema 和 signal output refs。 | event source/time/schema/alignment refs。 | event extension + signal refs。 | 缺 P0 refs 或 forbidden counter 非 0 -> blocked。 |
| ML adapter extension | 表达 training snapshot、feature/label/model/validation/prediction refs。 | ML metadata refs。 | ML extension + prediction signal refs。 | 缺 P0 refs 或 training/registry counter 非 0 -> blocked。 |
| Evidence typed refs | 将 adapter evidence 写成 refs-only typed records。 | adapter validation result、type-specific refs。 | evidence index typed refs。 | body_copy_count > 0 -> blocked。 |
| No-runtime guard | 汇总 forbidden operation counters 并 fail-closed。 | adapter operation counter report。 | pass / blocked decision。 | 任一 forbidden counter 非 0 -> blocked。 |

## 6. ADR 候选

| ADR | 决策 | 状态 | 备选 |
|---|---|---|---|
| ADR-CR158-001 | 采用 thin shared core + typed event/ML extensions。 | accepted-cp3 | fat common schema；two independent adapters |
| ADR-CR158-002 | Evidence extension 只存 typed refs 和短元数据，不复制正文。 | accepted-cp3 | adapter 自带 evidence body；full report embedding |
| ADR-CR158-003 | No-runtime guard 由 adapter counter report + FEAT-07 policy 双层执行。 | accepted-cp3 | 仅 CP7 文档审查；直接 runtime proof |

## 7. Use Case -> Architecture Traceability

| Use Case / Scenario | Architecture Coverage | Verification Mode |
|---|---|---|
| UC-58-CR158 step 1 | CR158 keeps one Change Package and one HLD/ADR set. | CP3 review |
| UC-58-CR158 step 2 / SC-CR158-P01 | Shared core + event/ML typed extensions. | contract/unit fixture |
| UC-58-CR158 step 3 / SC-CR158-P02 | AdapterTypedEvidenceRef stores refs-only metadata. | static / traceability |
| UC-58-CR158 step 4 / SC-CR158-N02 | AdapterOperationCounterReport + FEAT-07 guard. | security fixture |
| SC-CR158-N01 | Missing event/ML P0 refs produce machine-readable blocked reasons. | negative fixture |
| SC-CR158-B01 | CP2/CP3/CP5 gate sequence blocks direct implementation. | workflow/static |
| SC-CR158-A01 | Release wording uses fixture/static readiness only. | doc-review |

## 8. 场景模拟

| Simulation | 输入 | 预期架构路径 | 结果 |
|---|---|---|---|
| SIM-CR158-01 unified pass | event refs + ML refs 全部齐备，forbidden counters 全 0。 | event/ML extension -> shared core -> typed evidence -> no-runtime guard -> handoff refs。 | PASS |
| SIM-CR158-02 missing refs fail-closed | event 缺 alignment policy；ML 缺 validation report。 | extension validation 直接 blocked，写 machine-readable reason。 | PASS |
| SIM-CR158-03 runtime counter blocked | feed 或 training / registry counter 非 0。 | AdapterOperationCounterReport -> FEAT-07 guard -> blocked；不进入 handoff_ready。 | PASS |

## 9. 非功能设计

| 维度 | 设计承载 | 度量 |
|---|---|---|
| 可追溯性 | adapter refs -> evidence typed refs -> handoff refs。 | P0 refs 覆盖率 100%。 |
| 安全性 | forbidden operation counters + not-authorized wording。 | 禁用计数全部为 0。 |
| 可演进性 | shared core 与 type-specific extension 分离。 | event-only / ML-only 字段互不成为对方必填。 |
| 可测试性 | fixture/static positive、negative、boundary、doc-review 场景。 | P0 scenario planned coverage 100%。 |

## 10. HLD 拆分判定

| 判定信号 | 结论 | 理由 |
|---|---|---|
| 核心产物 > 1 | not split | event/ML adapters 是同一 unified adapter core 的两个 extension，不是两个独立产品。 |
| Story 数 > 5 | monitor | 候选 Story 约 5-6 个，但共享 core/evidence/guard 强耦合。 |
| ADR 分簇 | not split | 三个 ADR 共同约束 same adapter contract。 |
| 交付顺序独立 | not split | event/ML 必须一起验证 unified core 和 no-runtime guard。 |

结论：保持单份 HLD。若 CP5 发现 shared core 不成立，按 `DQ-CP3-CR158-001` 的回退条件拆分子 CR。

## 11. Tentative Story Groups After CP3 Approval

Story decomposition 仍需 CP3 批准后进入 CP4，本节只作为 CP4 输入。

| Candidate Story | Scope | Design Evidence Level |
|---|---|---|
| CR158-S01 | Shared adapter core and validation result contract。 | full-lld |
| CR158-S02 | Event strategy adapter typed extension。 | full-lld |
| CR158-S03 | ML strategy adapter typed extension。 | full-lld |
| CR158-S04 | Evidence index typed refs and Stage 2/3 handoff refs。 | full-lld |
| CR158-S05 | No-runtime guard counters and failure reasons。 | full-lld |
| CR158-S06 | Verification matrix and release wording boundary。 | technical-note |

## 12. Security and Authorization Boundary

CP3 approval does not authorize:

- source/test implementation before CP5 approval
- real event feed or live listener
- real ML model training, external model service call, model registry write or promotion
- real lake/NAS/provider access or credential/env/session read
- catalog/store/registry/model/prediction write
- QMT/MiniQMT/xtquant/gateway runtime
- simulation/paper/live/trading runtime or broker operation
- external framework clone/install/run
- Git remote write
- true release execution, publish, production deployment or runtime readiness claim

## 13. Risks and Mitigations

| Risk | Severity | Mitigation |
|---|---|---|
| R-CR158-ADAPTER-CONTRACT-COUPLING | medium | Thin core + typed extensions; CP5 must prove event-only and ML-only fields are not cross-required. |
| R-CR158-EVIDENCE-SEMANTIC-OVERFIT | medium | Evidence extension stores typed refs only; body_copy_count must remain 0. |
| R-CR158-RUNTIME-OVERCLAIM | high | HLD/ADR/CP3/CP8 wording repeats no-runtime boundary; counters must be 0. |

## 14. CP3 Decision Items

| Decision ID | Type | Question | Recommendation | Alternatives |
|---|---|---|---|---|
| DQ-CP3-CR158-001 | architecture | 是否确认 thin shared core + typed event/ML extensions 作为 CP4/CP5 输入？ | Confirm Option A。 | Fat common schema；two independent adapters。 |
| DQ-CP3-CR158-002 | security | 是否确认 CP3 不授权真实 runtime/data/registry/publish？ | Confirm fixture/static only。 | 只读真实数据授权；runtime authorization CR。 |

## 15. Gotchas

1. 不要把 shared core 扩成包含 event payload 或 ML model metadata 的大 schema。
2. 不要把 ML `model_artifact_ref` 写成 registry entry 或 promotion proof。
3. 不要把 event source ref 写成 live feed subscription。
4. 不要让 handoff consumer 直接读取 event/ML private fields。
5. 不要把 CP3 approval 写成 implementation 或 runtime authorization。

## 16. 自审

| 检查项 | 结果 | 证据 |
|---|---|---|
| 至少 2 个候选方案比较 | PASS | §3 advisor table 含 A/B/C |
| Architecture Gray Areas 前置 | PASS | §3 |
| Use Case -> Architecture Traceability | PASS | §7 |
| 关键场景模拟通过 | PASS | §8 |
| 拆分原则已应用 | PASS | §10 |
| 不授权边界明确 | PASS | §12 |
