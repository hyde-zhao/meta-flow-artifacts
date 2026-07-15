---
status: approved-cp3
version: "0.3"
change: CR-170
source_hld: process/archive/design-cr-docs/HLD-CANONICAL-RELIABILITY-NA-SEMANTICS-ADMISSION.md
---

# Canonical Reliability N/A Semantics and Admission Blueprint

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-15 | host-orchestrator inline meta-se-critical | 登记 CR-170 对 FEAT-14 canonical reliability/admission 能力的增量；不新增平行 Gate、runner 或 verifier Feature。 |
| 0.2 | 2026-07-15 | host-orchestrator inline meta-se-critical | 为 inventory 能力补充 baseline path/direction/disposition 与 15/5/1 分组；明确 fixture/test caller authoring 和 T3 零修改兼容边界。 |
| 0.3 | 2026-07-15 | host-orchestrator inline | 回填 CP3 已批准状态并迁入 `process/archive/design-cr-docs/`；当前 authority 继续由设计根目录索引维护。 |

## 能力地图

| Capability | Owner | 职责 | 非职责 | 输出 |
|---|---|---|---|---|
| CAP-CR170-01 Canonical N/A Policy Inventory | FEAT-14 | 维护 Gate 1-5 21-unit applicability/owner/boundary/result/baseline-path/direction/disposition contract | 不生产 evidence，不读真实数据 | 21 `NaPolicySpec`，方向 15/5/1 |
| CAP-CR170-02 Five-State Evidence Decision | FEAT-14 | 将 ref/reason/boundary 归一为五态 | 不把 generic reason 当 evidence | auditable decision |
| CAP-CR170-03 Gate Consumer Hardening | FEAT-14 | Gate 1-5 对 applicable mandatory negative state 生成 claim 与非 PASS | 不修改 CR-168/169 adapter | existing Gate summaries |
| CAP-CR170-04 Admission Tier Hardening | FEAT-14 / FEAT-07 | 保留 bottom-up merge，硬化 T0-T2 admission result，并回归保护现有 T3 early-return | 不接 runner，不启动 Stage3，不重写 T3 | existing AdmissionPolicyResult |
| CAP-CR170-05 Verification Contract | FEAT-14 / FEAT-08 | 当前 Gate 维护者自验证，向 future verifier 暴露审计 contract | 不实现 FU-006 verifier lane | policy/claim/status evidence refs |

## 跨能力流程

| Flow | 方向 | Owner | Fail-closed 条件 |
|---|---|---|---|
| FLOW-CR170-01 | evidence -> policy normalizer -> Gate 1-5 | FEAT-14 | unknown/missing/generic/incomplete 不得 PASS |
| FLOW-CR170-02 | Gate summaries -> protected merge -> admission resolver | FEAT-14 | mandatory NR 在 T1/T2 不得 PASS；T3 not authorized |
| FLOW-CR170-03 | canonical outputs -> adapter regressions / future verifier audit | FEAT-14 | adapter 不删除；verifier 未实现不得声称 independent |
| FLOW-CR170-04 | fixture/test caller -> explicit boundary/auth-ref -> evaluator | FEAT-14 | evaluator 不合成 boundary 或授权，不读取 credential |

## Feature 边界决策

CR-170 不新增 Feature。能力归入现有 FEAT-14 Mature Strategy Admission / Reliability Gate，安全授权表述由 FEAT-07 约束，文档/审计出口由 FEAT-08 消费。FEAT-03 研究 producer、FEAT-11 runner、FU-009 aggregate 与 FU-006 verifier 不被 CR-170 吞并。

## 自检

| 检查 | 结果 |
|---|---|
| 新平行 Gate/registry/envelope 数 | `0` |
| Feature owner 明确 | PASS，FEAT-14 主责 |
| runner/real-data/runtime 授权扩大 | `0` |
| future verifier 被误报为 current | `0` |
