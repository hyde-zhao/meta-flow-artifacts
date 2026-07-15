---
status: approved-cp3
version: "0.3"
change: CR-170
source_hld: process/archive/design-cr-docs/HLD-CANONICAL-RELIABILITY-NA-SEMANTICS-ADMISSION.md
---

# Canonical Reliability N/A Semantics and Admission Dependency Map

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-15 | host-orchestrator inline meta-se-critical | 冻结 Gate validators、policy evaluator、protected merge、admission resolver、adapters 与 future verifier 的单向依赖。 |
| 0.2 | 2026-07-15 | host-orchestrator inline meta-se-critical | 明确 fixture/test caller 对 boundary/auth-ref 的单向写入责任、evaluator 禁止合成授权引用，以及 T3 existing early-return 的零修改边界。 |
| 0.3 | 2026-07-15 | host-orchestrator inline | 回填 CP3 已批准状态并迁入设计归档；依赖方向和禁止依赖保持不变。 |

## 允许依赖

| From | To | 类型 | 约束 |
|---|---|---|---|
| Gate 1-5 validator | internal policy inventory/evaluator | call | 只消费同模块内部 five-state decision，不改变 public signature |
| internal evaluator | evidence mapping + normalized profile/strategy/claims | read | 不读 lake/provider/runtime，不调用 Gate validator |
| fixture/test caller | `n_a_boundaries` / static `authorization_ref` | explicit input | caller 负责 policy-specific 4/4 boundary；ref 仅为非敏感审计指针 |
| protected merge | existing `ReliabilityGateSummary` / `ArtifactRef` | aggregate | 只做 worst-state，不解释 tier |
| `resolve_admission_policy` | Gate 1-5 summaries | read/decision | 只做 tier/admission，不反写 summary |
| CR-168/169 adapters | canonical public Gate callable | call | 保留 adapter-local pre/post guards |
| FU-006 future verifier | public result + audit evidence refs | future read | 不依赖内部 `NaPolicySpec`/decision 类型 |

## 禁止依赖

| ID | From | To | 原因 | 替代路径 |
|---|---|---|---|---|
| FD-CR170-01 | policy evaluator | global runtime/lake/provider/NAS/credential | CR-170 是 static hardening | 仅消费显式 mapping |
| FD-CR170-02 | Gate validators | generic `_has_na_reason` bool 作为 mandatory 充分条件 | 无法表达 boundary/owner/applicability | five-state evaluator |
| FD-CR170-03 | admission resolver | current Stage3 runner/aggregate | 不在本 CR 授权范围 | Stage3 Launch CR / FU-009 |
| FD-CR170-04 | canonical hardening | CR-168/169 adapter production mutation | adapter 仍承担 defense-in-depth | 仅回归；未来 ADR |
| FD-CR170-05 | future verifier | private helper/internal enum | 会耦合内部实现且 FU-006 未实现 | public result/audit contract |
| FD-CR170-06 | CP3 design | implementation/test execution | CP3 只冻结架构 | CP5 批准后实现 |
| FD-CR170-07 | policy evaluator | synthesized boundary/authorization decision/credential store | evaluator 不拥有授权，也不得补写 caller 输入 | 缺失即 five-state fail-closed；真实授权另行立项 |
| FD-CR170-08 | CR-170 hardening | T3 existing early-return production rewrite | 当前 T3 已正确 BLOCKED+NOT_AUTHORIZED | 仅做 1/1 兼容回归，失败才重开 design delta |

## 循环风险

| Cycle | 风险 | 处理 |
|---|---|---|
| CYCLE-CR170-01 | evaluator 反向调用 Gate 来判断 applicability | eliminated：applicability 由 inventory + context 决定 |
| CYCLE-CR170-02 | merge 解释 tier、resolver 又重聚合 Gate | eliminated：merge 只传状态，resolver 只做 tier |
| CYCLE-CR170-03 | canonical 与 adapter 互相依赖私有 helper | eliminated：adapter 只调 public callable，canonical 不依赖 adapter |
| CYCLE-CR170-04 | future verifier 迫使 production 暴露内部类型 | eliminated：verifier 消费公开输出与证据 refs |

## 自检

| 检查 | 结果 |
|---|---|
| public callable direction 单向 | PASS |
| private helper runtime dependency | `0` |
| runner/aggregate/new external dependency | `0` |
| synthesized boundary/authorization dependency | `0` |
| T3 production branch modification default | `0` |
| 循环依赖 | `0` |
