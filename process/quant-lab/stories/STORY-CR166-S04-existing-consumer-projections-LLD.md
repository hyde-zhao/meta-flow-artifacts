---
story_id: "CR166-S04-existing-consumer-projections"
title: "Existing-consumer projections and CR155 regression"
story_slug: "existing-consumer-projections"
lld_version: "0.1"
tier: "L"
status: "confirmed"
confirmed: true
created_by: "host-orchestrator inline meta-dev"
created_at: "2026-07-13T12:33:00+08:00"
confirmed_by: "user"
confirmed_at: "2026-07-13T13:43:35+08:00"
feature_design_refs: ["docs/features/walk-forward-oos-projections/DESIGN.md", "docs/features/walk-forward-oos-projections/TEST-PLAN.md", "docs/features/walk-forward-oos-projections/TASKS.md", "docs/features/walk-forward-oos-producer/DESIGN.md"]
lld_policy: {required_level: "full-lld", trigger_reasons: ["cross-module-integration", "backward-compatibility", "worse-state-merge", "claim-ceiling"], rationale: "Three policy owners must consume one C2 identity without status improvement."}
open_items: 0
---

# LLD: CR166-S04 — Existing-consumer Projections

## 0. 上游设计依据（工程依据）

CR166 HLD §6.2、ADR-007/010、Domain RULE-166-11/12/16、FEAT-166-04 三件套、S03 self-validated component。

## 1. 目标（Goal）

新增 mapping-only projection module，并最小修改 CR151 statistical gate、CR154 Gate 2 和 StrategyAdmissionPackage，使三者消费同一 ref/hash/availability/reasons，保持原 policy ownership 与 claim ceiling。

## 2. 需求（Requirements）

consumer=3/3；identity difference=0；worse-only status improvement=0；new gate=0；raw fold recomputation=0；CR155 paper promotion=0；runtime auth flags 4/4 unchanged。

## 3. 模块拆分与职责

| 模块 | 职责 | 不负责 |
|---|---|---|
| `walk_forward_oos_projections` | self-validation guard、3 DTO mappings、identity equality | gate threshold/overall admission |
| statistical gate | consume compatible plan + identity/reasons | C2 production |
| reliability Gate 2 | consume split/wf/OOS/leakage mapping | raw fold recomputation |
| admission package | attach ref/reasons/limitations，status worse-only | runtime authorization |

## 4. 代码结构与文件影响

| 动作 | 文件 | 内容 |
|---|---|---|
| 新建 | `engine/walk_forward_oos_projections.py` | projection functions 与 shared identity guard。 |
| 修改 | `engine/strategy_admission_statistical_gate.py` | legacy plan compatible evidence identity/reasons；threshold owner 不变。 |
| 修改 | `engine/cross_strategy_reliability_gates.py` | Gate 2 typed mapping；现有 blocked claims 不删除。 |
| 修改 | `engine/strategy_admission_package.py` | C2 attach API/reason code/evidence ref；runtime flags 不变。 |
| 新建/修改 | 两个 projection/regression test 文件 | 3/3、worse-only、CR155。 |

## 5. 数据模型与持久化

Projection DTO 只复制 component identity、availability/outcome/reasons、fold summary 与 policy refs；不复制 raw mutable input。package 仍 immutable copy/replace 语义。无新存储、registry 或 schema migration。

## 6. API / Interface

| API | 时机 | 输出 / 失败 |
|---|---|---|
| `project_to_statistical_walk_forward_plan` | statistical build 前 | compatible plan；non-present 无虚假 fold metrics |
| `project_to_reliability_gate2` | Gate 2 evaluation 前 | existing evidence dict；N/A 仅按 strategy applicability |
| `attach_walk_forward_oos_evidence` | package build/attach | package copy；blocked/unavailable/fail/review 保守传播 |

## 7. 核心流程与状态映射

验证 component hash/ref → build three mappings → compare identity/reasons equality → existing consumers evaluate. blocked/typed_unavailable 生成 blocker；present fail/review 不可被其他 PASS 覆盖；present pass 只提供 eligibility，仍由各 consumer gate/policy 判断。event N/A 不满足 daily/ML mandatory C2。

## 8. 技术细节

CR151 legacy `fold_metrics[*].passed` 只能由 S03 outcome 派生；`_walk_forward_pass_rate` 可继续消费兼容数据，但不得成为 C2 producer。CR154 复用现有 ref fields/blocked claims。package 新 reason code 命名遵循 `MF_ADMISSION_*`，不清除既有 blocked reasons。

## 9. 安全与性能

projection 线性复制摘要，无 I/O/dereference。所有 not-authorized counters/flags 逐字段保留；C2 PASS 不能更改 paper/simulation/live/QMT/broker readiness。

## 10. 测试设计

present 3/3 identity；legacy pass-rate；Gate2 fields；package flags；blocked/unavailable/fail/review/pass lattice；mismatch/tamper；CR155 blocked/paper false；event N/A no component。

## 11. 实施步骤

T01 projection/guard；T02 statistical integration；T03 Gate2 integration；T04 package attach；T05 integration/CR155 regression。

## 12. 风险与 Gotchas

无 OPEN/LCQ。Gotchas：projection≠new gate；legacy passed 字段不是 caller truth；present pass≠overall pass；N/A≠mandatory satisfaction；不得 backfill CR155。

## 13. 回滚与发布

可按三 consumer 逐个移除 attach/mapping，C2 component 保留但不被准入消费；existing policies/CR155 恢复原状。无发布授权。

## 14. DoD / Definition of Done

- [x] 3/3 consumer identity/reasons 完全一致
- [x] status improvement/new gate/raw recomputation 均为 0
- [x] CR155 paper promotion=0；runtime flags 4/4 unchanged
- [x] CP5 已批准，`confirmed=true`；允许进入 repository-local 实现
