---
checkpoint_id: "CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-SCOPE"
checkpoint_name: "CR157 Stage 2 Multifactor Research Framework Upgrade Scope Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-05T11:45:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-05T12:01:53+08:00"
auto_check_result: "process/checks/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-SCOPE.result.json"
target:
  phase: "requirement-clarification"
  cr_id: "CR-157"
  artifacts:
    - "process/changes/CR-157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-2026-07-05.md"
    - "process/context/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-CONTEXT.yaml"
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
    - "docs/product/RELEASE-SLICES.md"
    - "docs/product/BACKLOG.md"
---

# CP2 CR157 Stage 2 Multifactor Research Framework Upgrade Scope Baseline 人工审查

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CR157 formal CR exists | PASS | `process/changes/CR-157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-2026-07-05.md` | `lifecycle_status=active`，`gate_status=cp2_pending`。 |
| CP0 request intake passed | PASS | `process/checks/CP0-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-REQUEST-INTAKE.result.json` | Stage 2 selected; FU-CR154-001 confirmed closed. |
| CP1 use-case completeness passed | PASS | `process/checks/CP1-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-USE-CASE-COMPLETENESS.result.json` | Product baseline drafts and SGQ routing exist. |
| Context capsule structurally complete | PASS | `process/context/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-CONTEXT.yaml` | Capsule includes must_read, read_if_needed, do_not_read_by_default, token_control, key_facts, risks_and_decisions, handoff, read_expansion_log, validation and allowed_reads. |
| Required user scenario confirmation | PASS | `process/discussions/CP2-CR157-SCENARIO-DISCUSSION-LOG.md` | 用户回复 `approve`，`SGQ-CR157-001` resolved。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | CR157 first slice 是否限定为 mature admission package builder、research evidence traceability、Stage 2/Stage 3 handoff hardening 和 no-runtime guard | PASS | `DQ-CP2-CR157-FIRST-SLICE` | 用户已批准推荐范围。 |
| 2 | event/ML adapter implementation 是否从本轮 first slice 延后 | PASS | `docs/product/MVP-SCOPE.md#Deferred` | 用户已批准延后到 backlog / 后续 CR。 |
| 3 | 是否继续禁止 lake write、NAS、provider、credential、QMT/gateway、simulation/live/trading、publish | PASS | `process/context/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-CONTEXT.yaml#risks_and_decisions` | 继续禁止。 |
| 4 | CP2 approval 是否仅授权进入 CP3 solution-design | PASS | `process/context/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-CONTEXT.yaml#handoff` | 不授权 implementation 或 runtime。 |
| 5 | Decision Collection Coverage 是否完整 | PASS | 下方 `### Decision Collection Coverage` | 已扫描 8 个来源，发现候选问题 6 个，纳入待决策 1 个。 |
| 6 | CP2 特定 Decision Brief 内容是否完整 | PASS | 下方 `### CP2 特定内容` | 已覆盖用户真实意图、场景覆盖摘要、范围取舍、认知盲区、Scenario Gray Areas、Deferred Ideas、用户选择影响和回退方式。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP2 auto precheck | PASS | `process/checks/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-SCOPE.result.json` | 用户批准后已无 blocker。 |
| Pending decisions collected | PASS | 下方 Decision Brief | 1 项必须用户决策。 |
| Unauthorized scope explicit | PASS | 下方审批者摘要 | CP2 approval 不授权真实外部系统、runtime、交易或发布。 |
| Next route clear | PASS | `next_route_if_approved=CP3` | 批准后只进入 solution-design。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Product use cases | `docs/product/USE-CASES.md` | draft | UC-58 CR157 增量用例。 |
| Product requirements | `docs/product/REQUIREMENTS.md` | draft | REQ-CR157-*。 |
| Scenarios | `docs/product/SCENARIOS.yaml` | draft | P0 正向、负向、边界和 precheck 场景。 |
| Test matrix | `docs/product/TEST-MATRIX.md` | draft | 覆盖矩阵。 |
| Story map | `docs/product/STORY-MAP.md` | draft | 候选 Story Map，非 dev-ready。 |
| MVP scope | `docs/product/MVP-SCOPE.md` | draft | In scope / out of scope / deferred。 |
| Release slices | `docs/product/RELEASE-SLICES.md` | draft | CP2 到 CP8 候选切片。 |
| Backlog | `docs/product/BACKLOG.md` | draft | event/ML adapters 和后续 observation workflow。 |
| Discussion checkpoint | `process/checks/CP2-CR157-DISCUSSION-CHECKPOINT.json` | draft | SGQ 状态机器恢复点。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR157 Stage 2 多因子研究框架升级的产品范围基线，使后续 CP3 只围绕明确的 mature admission package builder、evidence traceability、handoff hardening 和 no-runtime guard 展开。 |
| 推荐动作 | `approve`：批准 CR157 first slice 只覆盖多因子 Stage 2 framework deepening；event/ML adapter implementation 延后到 backlog 或后续 CR。 |
| approve 后会发生什么 | CR157 进入 CP3 solution-design；下一步设计 mature admission package builder 合同、Stage 2/Stage 3 handoff、evidence index 和 no-runtime guard。 |
| approve 不授权什么 | 不授权实现、LLD、真实 lake write、NAS/provider/credential、QMT/MiniQMT/xtquant/gateway runtime、simulation/paper/live/trading/broker、catalog/store/registry write、external framework、Git remote、publish 或 true release execution。 |
| 不确认会阻塞什么 | 阻塞 CR157 的 CP3 设计、Story 拆分、LLD、实现、验证和 release readiness。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR157-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE-CONTEXT.yaml` |
| capsule 状态 | `ready`，`DQ-CP2-CR157-FIRST-SLICE` 已由用户 `approve`。 |
| read_profile | `compact` |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。 |
| 全文档读取扩展 | 5 次：formal CR、component baseline、Stage 2 no-lake initial slice、scenario discussion log、discussion checkpoint；均已进入 `READ-EXPANSION-LEDGER.ndjson` 或本 capsule `read_expansion_log`。 |
| 缺失 / waived 理由 | 无缺失来源；CP3/CP4/CP5/CP6/CP7/CP8 证据在 CP2 前 N/A。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json`、`process/current/CURRENT.json` | scanned | 1 | 1 | `pending_gate=CP2-CR157...`，指向 `DQ-CP2-CR157-FIRST-SLICE`。 |
| GATE ledger | `process/state/GATE-LEDGER.ndjson` | scanned | 1 | 1 | CP2 gate opened，pending decision IDs 包含 `DQ-CP2-CR157-FIRST-SLICE`。 |
| 委托 Agent 交还摘要 | N/A | n/a | 0 | 0 | 本轮未真实 spawn meta-pm；由 host-orchestrator inline 产出产品基线，状态中已记录原因。 |
| 自动预检结果 | `process/checks/CP1-...result.json`、`process/checks/CP2-...result.json` | scanned | 1 | 1 | CP2 result blocker 为 `SGQ-CR157-001`。 |
| discussion log / checkpoint | `process/discussions/CP2-CR157-SCENARIO-DISCUSSION-LOG.md`、`process/checks/CP2-CR157-DISCUSSION-CHECKPOINT.json` | scanned | 3 | 1 | SGQ-CR157-001 待决策；SGQ-CR157-002/003 已由 CR 正文和门禁规则 resolved。 |
| 下游正式产物 | HLD / LLD / REVIEW / FIXES / release docs | n/a | 0 | 0 | CP2 未批准前不得产生 CP3+ 下游产物。 |
| 用户显式选择题 | 当前用户评审意见、CR157 formal CR | scanned | 1 | 1 | 用户要求判断哪些评审问题需要整改；范围建议为 approve，但合规要求先修 Decision Brief / capsule。 |
| 产品基线文档 | `docs/product/USE-CASES.md`、`REQUIREMENTS.md`、`SCENARIOS.yaml`、`TEST-MATRIX.md`、`MVP-SCOPE.md`、`BACKLOG.md` | scanned | 0 | 0 | 文档中无新增未决人工决策；event/ML adapter 已作为 deferred/backlog。 |

决策收集覆盖摘要：已扫描 8 个来源，发现候选问题 7 个，纳入待决策 1 个；N/A 来源 2 个，原因见上表。

### CP2 特定内容

| 字段 | 内容 |
|---|---|
| 用户真实意图 | 用户要求从 CR156 收敛点推进到 Stage 2 多因子研究框架升级，同时避免重开已关闭的 FU-CR154-001。 |
| 场景覆盖摘要 | `SCENARIOS.yaml` 当前包含 6 个场景：2 个正向、2 个负向、1 个边界、1 个 precheck；P0 场景 5 个，P0 planned coverage 5/5。 |
| `SCENARIOS.yaml` / `TEST-MATRIX.md` 覆盖摘要 | P0 requirements 5 个、P0 scenarios 5 个、P0 planned coverage 5 个；外部 runtime/provider/NAS/credential/trading 授权测试计数均为 0。 |
| `STORY-MAP.md` / `MVP-SCOPE.md` / `RELEASE-SLICES.md` 范围取舍 | 推荐 first slice 包含 mature admission package builder、research evidence traceability、Stage 2/Stage 3 handoff hardening、no-runtime guard；event/ML adapter implementation 与 Stage 4 observation workflow 延后。 |
| 认知盲区 | 当前 Stage 2 出口条件中的 FactorSpec、FactorRunSpec、factor panel、label window、evaluation、portfolio/risk 层需在 CP3 明确引用 CR150/CR151/CR154/CR155 的已满足证据，避免 CR157 被误解为重新覆盖全链路合同。 |
| Scenario Gray Areas | `SGQ-CR157-001` 已由用户 `approve`；`SGQ-CR157-002` 不授权 runtime/外部系统已确认；`SGQ-CR157-003` CP2 前不得进入 HLD/Story/LLD/实现已确认。 |
| Deferred Ideas | `DF-CR157-001` event strategy adapter、`DF-CR157-002` ML strategy adapter、`DF-CR157-003` Stage 4 observation review workflow。 |
| 用户选择影响 | 选择推荐方案会保持 CP3 架构边界窄且可验证；选择加入 event/ML adapters 会扩大 Feature owner、Story 数、测试矩阵和 adapter 合同耦合；选择只做文档会降低当前 CR 的框架深化价值。 |
| 回退方式 | CP2 前可直接修改产品基线和 Decision Brief；CP2 approved 后若要扩大范围，回到 CP2 或另起 follow-up CR；若发现 event/ML adapters 是当前 release blocker，则在 CP3 前回退到 CP2 扩范围。 |
| discussion log / checkpoint | `process/discussions/CP2-CR157-SCENARIO-DISCUSSION-LOG.md`、`process/checks/CP2-CR157-DISCUSSION-CHECKPOINT.json`。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 1 | `DQ-CP2-CR157-FIRST-SLICE` 需要用户确认。 |
| 高风险策略确认 | 1 | no-runtime / no-provider / no-trading / no-publish 边界继续保持。 |
| agent 默认处理 | 3 | 字段命名、draft 文档措辞和后续 CP3 设计细节可由 agent 默认收敛。 |
| 仅审计记录 | 3 | CR156 retrospective、Stage 2 no-lake initial slice、现有 component doc 作为输入，不重开旧 CR。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR157-FIRST-SLICE | scope | CR157 first slice 是否只覆盖多因子 mature admission package builder、research evidence traceability、Stage 2/Stage 3 handoff hardening 和 no-runtime guard，并延后 event/ML adapters？ | 批准推荐范围；event/ML adapter implementation 作为 backlog / 后续 CR。 | A. 同时纳入 event adapter；B. 同时纳入 event 和 ML adapters；C. 只做文档不做后续实现。 | 推荐方案最小且贴合当前 Stage 2 收敛点；A/B 价值更大但会扩大架构和 Story blast radius；C 风险最低但无法推进框架深化。 | 决定 CP3 设计边界、Story 拆分和验证范围。 | 若后续发现 event/ML adapters 是当前 release blocker，则另起 CR 或在 CP3 前回到 CP2 扩范围。 |

### 用户需决策事项

| 字段 | 内容 |
|---|---|
| 本轮必须决策数量 | 1 |
| 用户已回复 | `approve` |
| `approve` 表示 | 接受 `DQ-CP2-CR157-FIRST-SLICE` 的推荐方案，即 CR157 first slice 只覆盖多因子 mature admission package builder、research evidence traceability、Stage 2/Stage 3 handoff hardening、no-runtime guard 和 adapter backlog alignment；event/ML adapter implementation 延后。 |
| `approve` 不表示 | 不授权实现、LLD、真实 lake write、NAS/provider/credential、QMT/MiniQMT/xtquant/gateway runtime、simulation/paper/live/trading/broker、catalog/store/registry write、external framework、Git remote、publish 或 true release execution。 |
| 若后续选择修改 | 必须回到 CP2 或另起 CR，说明要修改 `DQ-CP2-CR157-FIRST-SLICE` 的范围，例如加入 event adapter、加入 event+ML adapters，或降级为只做文档。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-05T12:01:53+08:00
- 修改意见：无；用户回复 `approve`。
- 已接受决策项：`DQ-CP2-CR157-FIRST-SLICE`
- 风险接受项：接受 CR157 first slice 不纳入 event/ML adapter implementation；这些进入 backlog / 后续 CR。
- 不授权项：不授权实现、LLD、真实 lake write、NAS/provider/credential、QMT/MiniQMT/xtquant/gateway runtime、simulation/paper/live/trading/broker、catalog/store/registry write、external framework、Git remote、publish 或 true release execution。
