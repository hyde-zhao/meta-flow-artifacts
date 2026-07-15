---
checkpoint: CP2
cr_id: CR-170
title: "Canonical Cross-Strategy Reliability N/A Semantics and Admission Hardening — 需求、场景与范围基线"
status: approved
gate_profile: architecture-major
automatic_precheck: PASS
context_ref: process/context/CP2-CR170-REQUIREMENT-CONTEXT.yaml
result_ref: process/checks/CP2-CR170-REQUIREMENTS-BASELINE.result.json
decision_brief_profile: compact
opened_at: "2026-07-15T12:38:00+08:00"
opened_by: host-orchestrator
revision: 2
reviewed_by: "user"
reviewed_at: "2026-07-15T12:55:40+08:00"
approval_source: "conversation"
---

# CP2 — CR170 需求 / 场景 / 范围基线人工门禁

自动预检结论：`PASS`；人工审查状态：`approved`。本门禁只解锁 **CP3 solution-design**；不得直接进入 Story、LLD、实现或验证。

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 为 canonical Gate 1-5 冻结统一且可审计的 N/A evidence 语义，并保护/验证 Gate 6 的底层 worst-state merge、最小硬化 admission tier，杜绝 mandatory unresolved evidence 的无条件 PASS。 |
| 两个评审关注点的处置 | `build_shared_gate_summary` 已正确传播 NEEDS_REVIEW，设为受保护回归；`resolve_admission_policy` 是独立缺口，CP3 必须冻结两层边界。 |
| 推荐动作 | `approve`：接受下方五项推荐方案，只进入 CP3 设计五态代码映射、Gate1-5 consumption、protected merge 与 admission policy。 |
| approve 后会发生什么 | 回填 CP2 与 gate ledger，按 route plan 进行 HLD/ADR，推进到 CP3 人工门禁；仍不写生产代码。 |
| approve 不授权什么 | 不授权真实数据、current Stage3 runner 接入、historical run revalidation、aggregate、CR155 promotion、runtime/trading、发布或 Git remote write。 |
| 不确认会阻塞什么 | CR170 保持 awaiting-user/not_ready；HLD/CP3、Story、LLD、实现和验证全部阻塞。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR170-REQUIREMENT-CONTEXT.yaml` |
| capsule 状态 | awaiting-user / ready-for-human-gate |
| read_profile | compact |
| 默认读取策略 | 先读 CR summary、CP0/CP1 result、产品证据与本 checkpoint；只在字段冲突或深度审查时扩展正式 CR/产品文档/相关代码。 |
| 全文档读取扩展 | 3 次：正式 CR 人工审计、MULTIFACTOR 历史 Stage3 字段冲突、CP2 discussion 人工审计；均已写 READ-EXPANSION-LEDGER。 |
| 缺失 / waived 理由 | N/A；CP2 必要输入齐备。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| 用户评审输入 | 当前对话与 `process/discussions/CP2-CR170-SCENARIO-DISCUSSION-LOG.md` | scanned | 2 | 2 | Gate6 已有 merge 与 resolve admission 边界，合并到 DQ-BOUNDARY。 |
| 正式 CR / product scope | `process/changes/CR-170.md` / `docs/product/MVP-SCOPE.md` | scanned | 3 | 3 | scope、tier、adapter 边界进入 DQ。 |
| Requirements / scenarios | `docs/product/REQUIREMENTS.md` / `SCENARIOS.yaml` | scanned | 5 | 3 | 五态、tier、Gate1 三层断言；测试细节留 CP3/CP5。 |
| CP0 code probe | `process/checks/CR170-REMOTE-BASELINE-CONFLICT-PRECHECK.json` | scanned | 4 | 2 | merge/admission 事实与 Gate1-5 escape 纳入。 |
| Follow-up tracking | `process/changes/CR-161-FOLLOW-UP-TRACKING-2026-07-10.md` | scanned | 2 | 1 | adapter 简化和 aggregate 归 FU009。 |
| 授权策略 | CR170 `not_authorized_by_approve` | scanned | 6 groups | 0 | 全部为 deny-by-default，不是可由本轮 approve 放行的选项。 |

> 决策收集覆盖摘要：扫描 6 类来源、22 个候选问题/约束分组，去重后形成 5 项待人工决策；其余实现细节进入 CP3/CP5，未授权项固定为范围外。

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 5 | scope、五态、tier、Gate6 两层边界、adapter retention。 |
| 高风险策略确认 | 3 | T1/T2 fail-closed、禁止重写正确 merge、禁止移除局部 guard。 |
| agent 默认处理 | 4 | 21-unit inventory 文档化、reason-code/数据结构候选、测试 fixture 组织、兼容 diff 检查；具体形态由 CP3/CP5 审查。 |
| 仅审计记录 | 4 | CP0 probe、Stage2 7/7、当前 runner 无 caller、三类基线修正。 |

### 产品范围摘要

In Scope：

1. Gate 1-5 `21/21` N/A policy units inventory 与五态业务语义。
2. Gate 1 multiple-testing/FDR masked escape 的字段判定、mandatory claim、最终 worst-state `3/3` 断言。
3. Gate 2-5 mandatory missing、generic reason escape、incomplete boundary 的 fail-closed consumption。
4. 验证并保护 `build_shared_gate_summary` 现有 NEEDS_REVIEW 传播；无失败证据时修改数=`0`。
5. 在 `resolve_admission_policy` 或 CP3 选定的最小公共边界执行 T0/T1/T2/T3 policy。
6. 保持公共 API，并回归 CR168/CR169 adapter `2/2`。
7. 增量纠正 CR169 closed、C4/Stage2 7-of-7、legacy Stage3 marker 三类基线偏差。

Out of Scope：

- current Stage 3 runner 接线、历史 Stage 3 run 深度 revalidation 或 Stage 3 Launch。
- C1-C4 aggregate、mature StrategyAdmissionPackage 最终集成、FU-CR161-009 或 CR155 promotion。
- 删除/简化 CR168/CR169 adapter guard。
- 真实 lake/NAS/provider/credential/evidence、runtime、broker/QMT、simulation/paper/live/trading。
- catalog/store/registry 写入、publish/deploy/tag/release/Git remote write。

### 量化成功标准与 Claim Ceiling

| 项目 | 目标值 |
|---|---:|
| Use Case / Requirements / QAC | 1 / 9 / 15 |
| CR170 scenarios / matrix | 20 / 20 |
| P0 / P1 | 19 / 1 |
| Gate 1-5 policy inventory | 21/21 |
| 业务语义 | 5/5 |
| Gate 1 masked-escape assertion layers | 3/3 |
| tier outcomes | 4/4：T0 NR、T1/T2 BLOCKED、T3 NOT_AUTHORIZED |
| mandatory unresolved evidence 的 unconditional PASS | 0 |
| lower-level merge 无失败证据重写 | 0 |
| adapter regression | 2/2 |
| public incompatible break | 0 |
| runner/aggregate/real-data/runtime/remote-write/CR155 promotion | 0/0/0/0/0/0 |

用户/consumer 边界：独立验证者是 `future consumer`，由 `FU-CR161-006` 实现；CR170 当前验证由可靠性 Gate 维护者自验证代行，independent verifier lane 实现数=`0`，不得声称 verifier independence 已交付。

Claim ceiling：`stage2_complete=true`、Stage2 exit=`passed_7_of_7`、`stage3_started=false`、`stage3_entry_ready=false`；CP7 实际完成前 `canonical_gate_1_5_na_semantics_hardened=false`、`gate6_admission_needs_review_hardened=false`。

### 风险和未决问题

| 风险 | 控制 | 状态 |
|---|---|---|
| 改全局 helper 误伤合法 N/A | CP2 只冻业务语义；CP3 比较局部 policy evaluator/typed result，不预设改 `_has_na_reason` | OPEN-CP3 |
| 重写已有正确 Gate6 merge 扩大爆炸半径 | protected regression；失败证据前生产修改=0 | CONTROLLED-PENDING-TEST |
| admission tier 风险强度写反 | 固定 T0<T1<T2<T3；四 tier 分别测试 | CONTROLLED-PENDING-CP2 |
| Gate1 escape 被其他 claim 掩盖 | 三层断言，不只断言最终 status | CONTROLLED-PENDING-DESIGN |
| adapter 双层 guard 长期耦合 | CR170 保留；FU009 四条件+ADR 后再评估 | ACCEPTED-FOLLOW-UP |
| 历史 Stage3 claim 被误读 | 保留历史但标 legacy/require-revalidation；Launch CR 独立处理 | OPEN-OUT-OF-SCOPE |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR170-SCOPE | scope | hardening 范围是否一次覆盖 Gate1-5？ | Gate1-5 N/A semantics + Gate6 admission；不缩为 Gate4-only。 | 只 Gate4，其余另起 CR。 | 推荐避免半修复 canonical 状态和重复打开公共模块；备选爆炸半径小但会保留已 probe 的 Gate2/3/5 逃逸。 | 范围扩大到五个 Gate，但仍是同一公共语义与同一审计目标。 | 只有 21-unit inventory 证明其他 Gate 不存在 mandatory escape 时才缩小。 |
| DQ-CP2-CR170-SEMANTICS | architecture | 五态在 CP2 冻结到什么程度？ | 冻结业务语义，代码 enum/dataclass/helper 留 CP3。 | CP2 直接冻结代码形态。 | 推荐保留最小实现选择空间；备选更早确定但可能误改共享 `_has_na_reason`。 | 影响公共 schema/API 兼容性和合法 N/A 的误伤概率。 | 若 CP3 发现已有 public schema 必须固定类型，回填 ADR 并重开相关设计门。 |
| DQ-CP2-CR170-TIER | risk_acceptance | mandatory NEEDS_REVIEW 如何按 tier 处理？ | T0=NEEDS_REVIEW/no-PASS；T1/T2=BLOCKED；T3=NOT_AUTHORIZED。 | 所有 tier 一律 BLOCKED。 | 推荐匹配 T0<T1<T2<T3 风险强度并保留诊断；备选最保守但会阻断 exploratory 分析。 | 错配会导致弱 tier 比 release 更严，或 admission/release 虚假 PASS。 | 若 T0 诊断结果被下游误用为 admission，切换全 blocked 并重开 CP3。 |
| DQ-CP2-CR170-GATE6-BOUNDARY | architecture | Gate6 是否重写？ | 保留/回归 bottom-up merge，单独硬化 `resolve_admission_policy`。 | 重写 merge + admission。 | 推荐复用已验证逻辑、爆炸半径最小；备选统一但会重写正确传播。 | 影响所有 Gate summary 与 admission caller；错误重写可能丢失 worst-state。 | 仅在 `build_shared_gate_summary` regression 失败且有最小复现时扩大。 |
| DQ-CP2-CR170-ADAPTERS | follow_up_tracking | CR168/169 guard 如何处理？ | 本 CR 保留为 defense-in-depth；FU009 满足四条件后评估简化。 | 本 CR 同时删除。 | 推荐维持局部 containment 与公共 hardening 双层防线；备选代码少但可能降低 fail-closed。 | 影响现有 C3-only/C3+C4 caller 和未来 aggregate 耦合。 | 只有 caller 全经新 contract、删除不降 fail-closed、全回归 PASS、ADR 批准时切换。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 本轮待人工决策项 | 5 |
| 必须用户决策 | `DQ-CP2-CR170-SCOPE`、`DQ-CP2-CR170-SEMANTICS`、`DQ-CP2-CR170-TIER`、`DQ-CP2-CR170-GATE6-BOUNDARY`、`DQ-CP2-CR170-ADAPTERS` |
| 推荐回复 | `approve` |
| 如果你回复 approve | 同时接受上述五项推荐方案，并且只解锁 CP3 solution-design。 |
| 不表示授权 | 不表示授权 HLD 之外的实现、真实数据、Stage3、aggregate、CR155 promotion、runtime/trading、发布或远端写入。 |
| 修改 | 回复 `修改: <具体修改点>`，回退产品基线整改并重跑 CP1/CP2。 |
| reject | CR170 保持 requirement-clarification，不进入 CP3。 |

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP0 双仓基线 / 冲突预检 | PASS | `process/checks/CP0-CR170-BASELINE.result.json` |
| CP1 产品基线完整性 | PASS | `process/checks/CP1-CR170-USE-CASE-COMPLETENESS.result.json` |
| CP2 自动预检 | PASS | `process/checks/CP2-CR170-REQUIREMENTS-BASELINE.result.json` |
| Decision Brief / capsule | PASS | 本文与 `process/context/CP2-CR170-REQUIREMENT-CONTEXT.yaml` |

## Checklist

| # | 检查项 | 审查结果 | 证据 / 审查意见 |
|---:|---|---|---|
| 1 | 接受 Gate1-5 + Gate6 admission 的范围 | ☑ | DQ SCOPE；user approved |
| 2 | 接受五态先冻结业务语义、CP3 决定代码形态 | ☑ | DQ SEMANTICS；user approved |
| 3 | 接受 T0/T1/T2/T3 tier policy | ☑ | DQ TIER；user approved |
| 4 | 接受 protected merge + separate admission hardening | ☑ | DQ GATE6-BOUNDARY；user approved |
| 5 | 接受 adapter 保留与 FU009 简化条件 | ☑ | DQ ADAPTERS；user approved |
| 6 | 接受 9 requirements、15 QAC、20 scenarios、21/21 inventory | ☑ | CP1/CP2 自动结果 |
| 7 | 确认 Stage3/real-data/aggregate/CR155/runtime/remote-write 不授权 | ☑ | Claim ceiling / Out of Scope |
| 8 | 独立验证者仅为 future consumer，当前由 Gate 维护者自验证 | ☑ | UC-58 v1.8；FU-CR161-006 |

## Exit Criteria

- 用户对五项 DQ 给出 `approve`、`修改:` 或 `reject`。
- `approve` 只允许进入 CP3 solution-design；其他不授权项持续生效。
- 若 `修改:`，更新产品基线并重新运行 CP1/CP2 自动检查。

## Deliverables

- CP0：`process/checks/CP0-CR170-BASELINE.result.json`
- CP1：`process/checks/CP1-CR170-USE-CASE-COMPLETENESS.result.json`
- 产品证据：`process/checks/CR170-PRODUCT-BASELINE-EVIDENCE.json`
- CP2 capsule：`process/context/CP2-CR170-REQUIREMENT-CONTEXT.yaml`
- 本 checklist：`process/checkpoints/CP2-CR170-REQUIREMENTS-BASELINE.md`

## 人工审查结果

- 状态：`approved`
- 审批人：用户
- 时间：`2026-07-15T12:55:40+08:00`
- 已接受决策：`DQ-CP2-CR170-SCOPE`、`DQ-CP2-CR170-SEMANTICS`、`DQ-CP2-CR170-TIER`、`DQ-CP2-CR170-GATE6-BOUNDARY`、`DQ-CP2-CR170-ADAPTERS`
- 补强条件：独立验证者为 `future consumer`；CR170 当前由 Gate 维护者自验证；新增 consumer 调用视角但不改变量化分母。
- 备注：只批准进入 CP3 solution-design；Story、LLD、实现、验证、真实数据、Stage3、aggregate、CR155 promotion、runtime/trading 与 Git remote write 均未授权。
