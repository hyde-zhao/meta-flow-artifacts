---
checkpoint_id: "CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-SCOPE"
checkpoint_name: "CR158 Event + ML Strategy Adapter Scope"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-05T16:25:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-05T16:55:00+08:00"
auto_check_result: "process/checks/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-SCOPE.result.json"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "docs/product/USE-CASES.md"
    - "docs/product/REQUIREMENTS.md"
    - "docs/product/SCENARIOS.yaml"
    - "docs/product/TEST-MATRIX.md"
    - "docs/product/STORY-MAP.md"
    - "docs/product/MVP-SCOPE.md"
    - "docs/product/RELEASE-SLICES.md"
    - "docs/product/BACKLOG.md"
---

# CP2 CR158 Event + ML Strategy Adapter Scope 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP0-CR158-EVENT-ML-STRATEGY-ADAPTER-REQUEST-INTAKE.result.json` | PASS | 0 | CR158 intake completed; DF-CR157-001 and DF-CR157-002 merged into one formal CR. |
| `process/checks/CP1-CR158-EVENT-ML-STRATEGY-ADAPTER-USE-CASE-COMPLETENESS.result.json` | PASS | 0 | Product baseline docs and coverage matrix were updated for CR158. |
| `process/checks/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-SCOPE.result.json` | PASS | 0 | CP2 Decision Brief is ready for user review. |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR158 是否以一个统一 event + ML strategy adapter CR 继续，先完成 HLD / ADR 和 Story 设计批次，再进入本地 fixture/static 实现与验证。 |
| 推荐动作 | `approve` 推荐方案：统一 event + ML adapter scope，shared adapter core + type-specific extensions，保持 no-runtime/no-publish 授权边界。 |
| approve 后会发生什么 | CR158 进入 `solution-design`；由 meta-se 准备 HLD / ADR，之后才允许 Story decomposition。CP5 全量 Story 设计证据批准前不得实现。 |
| approve 不授权什么 | 不授权真实 event feed/live listener、真实 ML model training、external model service、model registry promotion、real lake/NAS/provider/credential access、catalog/store/registry/prediction write、QMT/gateway/runtime、simulation/paper/live/trading/broker、external framework run、Git remote write、publish 或 production deployment。 |
| 不确认会阻塞什么 | 阻塞 CR158 HLD、Story split、LLD、实现和验证；`DF-CR157-001/002` 保持 deferred/promoted-but-unapproved 状态。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档。 |
| 全文档读取扩展 | 0 次新增；本轮产品文档为当前 CP2 正式对象。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/state/STATE.current.json`, `process/state/GATE-LEDGER.ndjson` | scanned | 1 | 1 | 当前仅 `pending_gate=CP2`；纳入 `DQ-CP2-CR158-GATE-SEQUENCE`，用于确认 CP2 后不得绕过 CP3/CP5。 |
| CR / 用户显式请求 | 用户请求、`process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md` | scanned | 3 | 3 | 用户要求把 `DF-CR157-001` 与 `DF-CR157-002` 合并为一个 CR；CR frontmatter 同时暴露 no-runtime 与 gate sequence，纳入 3 个 DQ。 |
| CR157 deferred 来源 | `process/changes/CR-157-FOLLOW-UP-TRACKING-2026-07-05.md`, `docs/product/USE-CASES.md#CR158-Deferred-Promotion-Mapping` | scanned | 2 | 1 | 两个 deferred item 属同一 adapter delivery value；合并为 `DQ-CP2-CR158-UNIFIED-SCOPE`，不单独升级为两个用户决策。 |
| 产品用例与 SGQ 表 | `docs/product/USE-CASES.md#CR158-Scenario-Gray-Areas`, `docs/product/USE-CASES.md#CR158-用户可见场景确认证据` | scanned | 3 | 3 | SGQ-CR158-001/002/003 分别映射到统一 scope、no-runtime、安全边界和 gate sequence 决策。 |
| 需求与约束 | `docs/product/REQUIREMENTS.md` | scanned | 7 | 3 | REQ-CR158-001..007 已形成需求基线；需用户确认的是 scope、安全授权和推进顺序，其余为 agent 默认实现/验证约束。 |
| 自动预检结果 | `process/checks/CP0-CR158*.result.json`, `process/checks/CP1-CR158*.result.json`, `process/checks/CP2-CR158*.result.json` | scanned | 3 | 3 | 预检无 blocker；route profile、CP1 完整性和 CP2 scope readiness 均汇入本 Brief。 |
| SCENARIOS / TEST-MATRIX | `docs/product/SCENARIOS.yaml`, `docs/product/TEST-MATRIX.md` | scanned | 2 | 1 | 覆盖摘要支持 `DQ-CP2-CR158-NO-RUNTIME`；具体测试层级由后续 CP5/CP7 消费，不单独要求用户拍板。 |
| STORY-MAP / MVP-SCOPE / RELEASE-SLICES / BACKLOG | `docs/product/STORY-MAP.md`, `docs/product/MVP-SCOPE.md`, `docs/product/RELEASE-SLICES.md`, `docs/product/BACKLOG.md` | scanned | 4 | 2 | 范围取舍映射到 unified scope 和 gate sequence；Story 候选与 release slices 只作为 CP3/CP5 输入，不在 CP2 变成实现授权。 |
| discussion log / checkpoint | `process/discussions/CP2-CR158-SCENARIO-DISCUSSION-LOG.md`, `process/checks/CP2-CR158-DISCUSSION-CHECKPOINT.json` | missing | 0 | 0 | CR158 是用户直接启动的 CR157 deferred promotion；SGQ 证据已内联到 `USE-CASES.md` 与本 Brief。缺独立 discussion 文件不产生额外待决策项；若用户要求严格恢复点，可在 CP2 通过前补建。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮经用户批准采用 host-orchestrator inline fallback 完成 meta-pm 范围整理，证据在 `process/state/AGENT-DISPATCH-LEDGER.ndjson`。无独立 handoff summary。 |
| 下游正式产物 | HLD / ADR / LLD / REVIEW / release docs | n/a | 0 | 0 | CP2 未批准前不得生成下游设计、实现、验证或发布产物；不适用。 |
| **合计** | 11 类来源 | - | **25** | **3** | 其余 22 项归入 agent 默认处理、仅审计记录、下游 CP3/CP5/CP7 消费或 N/A。 |

### CP2 特定内容覆盖

| 必填项 | 覆盖摘要 | 证据 |
|---|---|---|
| 用户真实意图 | 用户要把 `DF-CR157-001` event adapter 与 `DF-CR157-002` ML adapter 合并为一个正式 CR，避免拆成两个互相冲突的 adapter contract 流程；当前批准只应推进 CP2 准备，不应被解释为已批准 CP2。 | 用户请求；`process/context/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-CONTEXT.yaml` objective.interpretation |
| 场景覆盖摘要 | CR158 新增 1 个用例、3 个 Scenario Gray Areas、3 条用户可见确认记录、6 条工程场景；P0 覆盖 unified core、typed evidence refs、missing refs fail-closed、no-runtime counter fail-closed、CP2 gate guard。 | `docs/product/USE-CASES.md`, `docs/product/SCENARIOS.yaml` |
| `SCENARIOS.yaml` 覆盖摘要 | 全量 12 条场景，其中 CR158 6 条；P0 总数 10，P0 planned coverage 10；CR158 覆盖 positive / negative / boundary / precheck。 | `docs/product/SCENARIOS.yaml#coverage_summary` |
| `TEST-MATRIX.md` 覆盖摘要 | CR158 6 条矩阵行全部 planned；验证模式为 fixture/static；真实 event feed、真实训练、registry、provider/NAS/credential、trading/simulation/live 授权计数均为 0。 | `docs/product/TEST-MATRIX.md#Coverage-Summary` |
| `STORY-MAP.md` 范围取舍 | CP2 阶段仅确认候选 Story：CR158-S01..S06；这些不是 `DEVELOPMENT-PLAN` 或 dev-ready Story。 | `docs/product/STORY-MAP.md#Activities` |
| `MVP-SCOPE.md` 范围取舍 | In Scope 是 unified adapter scope、event fixture/static contract、ML fixture/static contract、typed refs、no-runtime validation；Out of Scope 是真实 feed、训练、registry、provider/lake/NAS/credential、runtime/trading/publish。 | `docs/product/MVP-SCOPE.md` |
| `RELEASE-SLICES.md` 范围取舍 | Slice 0 仅为 CP2 product baseline；Slice 1 才是 adapter architecture；Slice 2 是设计批次；Slice 3 才是 fixture/static implementation；Slice 4 是 release readiness。 | `docs/product/RELEASE-SLICES.md#CR158-Candidate-Slices` |
| 认知盲区 | 共享 core 可能过度泛化；event 与 ML typed extension 可能互相污染；evidence index 可能从 refs-only 滑向复制大型正文；release wording 可能误导为 runtime-ready。 | `docs/product/USE-CASES.md#CR158-Scenario-Gray-Areas`, `process/changes/summaries/CR-158.summary.json` |
| Scenario Gray Areas | SGQ-CR158-001 统一 contract；SGQ-CR158-002 no-runtime/no-publish；SGQ-CR158-003 CP2 后不得直接实现。三项均进入待决策清单。 | `docs/product/USE-CASES.md#CR158-Scenario-Gray-Areas` |
| Deferred Ideas | CR157 的 `DF-CR157-001` / `DF-CR157-002` 已 promoted-active 到 CR158；真实 runtime、Stage 4 observation、publish、external framework run 仍 deferred / not authorized。 | `docs/product/USE-CASES.md#CR158-Deferred-Promotion-Mapping`, `docs/product/BACKLOG.md` |
| 用户选择影响 | 选择统一 scope 会减少重复门禁但要求 CP3 明确 shared core 与 type-specific extension；选择拆分会增加两个 CR 的 CP2/CP3/CP5 成本；选择只做 event 或 ML 会保留另一条 deferred debt。 | `DQ-CP2-CR158-UNIFIED-SCOPE` |
| 回退方式 | CP2 前可修改 Brief 或 reject；CP3 发现 shared core 不成立时拆分子 CR；任何真实 runtime 需求必须回退到 runtime authorization CR；只想试验时改为 Spike / experiment CR。 | 待人工决策清单 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 2 | `DQ-CP2-CR158-UNIFIED-SCOPE`, `DQ-CP2-CR158-GATE-SEQUENCE` 进入待人工决策清单。 |
| 高风险策略确认 | 1 | `DQ-CP2-CR158-NO-RUNTIME` 显式展示；`approve` 不授权真实 runtime / data / publish。 |
| agent 默认处理 | 16 | 产品文档修订记录、context、ledger、coverage matrix、refs-only 细节、后续 Story 候选整理，由 downstream agents 按门禁消费。 |
| 仅审计记录 | 6 | CR157 deferred history、promotion mapping、缺独立 handoff summary、下游产物 N/A、discussion standalone 文件缺失原因。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR158-UNIFIED-SCOPE | scope | CR158 是否把 event adapter 与 ML adapter 合并为一个统一 adapter CR？ | A. 合并为一个 CR，shared core + type-specific extensions。 | B. 拆成 event CR 与 ML CR；C. 只做 event；D. 只做 ML。 | A 减少重复 HLD/CP5 和 schema 分叉；B 降低单次范围但增加重复门禁和冲突；C/D 交付更小但会留下另一个 adapter 的后续债务。 | A 的主要风险是过度泛化 shared core；需要 CP3 明确 type-specific extension。 | 若 CP3 发现 event 与 ML contract 不可共用，拆分为两个子 CR 或调整 Story batch。 |
| DQ-CP2-CR158-NO-RUNTIME | security | CP2 approve 是否授权真实 feed、训练、registry、provider/lake/NAS/credential、runtime、trading或 publish？ | A. 不授权，只允许 local/static/fixture 设计、实现和验证。 | B. 授权只读真实数据；C. 授权真实 feed/training/runtime。 | A 与 CR157 边界一致，风险最低；B/C 会触发 runtime-high-risk，需要新授权、凭据/数据边界和更重验证。 | 若误写 runtime-ready 会造成 release overclaim 和安全边界破坏。 | 任何真实运行需求必须新建 runtime authorization CR 或回退 CP2。 |
| DQ-CP2-CR158-GATE-SEQUENCE | implementation | CP2 approve 后是否允许直接 Story/LLD/实现？ | A. 不允许；CP2 后进入 CP3 HLD/ADR，CP5 批准后才实现。 | B. 直接 Story split；C. 直接实现。 | A 符合 architecture-major route；B/C 更快但会绕过架构和设计证据，风险不可控。 | 直接实现会破坏 adapter contract、file ownership 和 no-runtime 验证设计。 | 若用户只想做 Spike，应取消当前 implementation CR 或改为 experiment/spike CR。 |

| 字段 | 内容 |
|---|---|
| 推荐决策 | `approve` 接受 `DQ-CP2-CR158-UNIFIED-SCOPE`、`DQ-CP2-CR158-NO-RUNTIME`、`DQ-CP2-CR158-GATE-SEQUENCE` 的推荐方案。 |
| 备选方案 | 可选择拆成 event / ML 两个 CR、只做 event、只做 ML、授权只读真实数据、授权真实 runtime、直接 Story split、直接实现、取消当前 implementation CR 或改为 Spike。 |
| 影响维度 | 用户价值、实现复杂度、可验证性、维护成本、平台兼容、安全 / 权限、交付影响。 |
| 优劣分析 | 统一 CR 降低合同分叉但需要 CP3 控制过度泛化；no-runtime 风险最低但不证明真实运行；CP3/CP5 顺序较慢但保留架构和设计证据。 |
| 风险与回退 | 风险包括 adapter contract 过度耦合、evidence semantic overfit、runtime overclaim；回退方式包括 CP2 修改 / reject、CP3 拆分子 CR、runtime authorization CR 或 Spike。 |
| 用户需决策事项 | 本轮用户只需确认 3 项：`DQ-CP2-CR158-UNIFIED-SCOPE`、`DQ-CP2-CR158-NO-RUNTIME`、`DQ-CP2-CR158-GATE-SEQUENCE`。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CR158 formal CR active | PASS | `process/changes/CR-158-EVENT-ML-STRATEGY-ADAPTER-UNIFIED-IMPLEMENTATION-2026-07-05.md` | `lifecycle_status=active`, `gate_status=cp2_pending`. |
| Product baseline docs updated | PASS | `docs/product/*` | USE-CASES / REQUIREMENTS / SCENARIOS / TEST-MATRIX / STORY-MAP / MVP-SCOPE / RELEASE-SLICES / BACKLOG updated. |
| CP1 completeness passed | PASS | `process/checks/CP1-CR158-EVENT-ML-STRATEGY-ADAPTER-USE-CASE-COMPLETENESS.result.json` | No blockers. |
| Context capsule ready | PASS | `process/context/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-CONTEXT.yaml` | Capsule-first context exists. |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | Scope includes both event and ML adapter and preserves CR157 history | PASS | `docs/product/USE-CASES.md`, `process/changes/CR-157-FOLLOW-UP-TRACKING-2026-07-05.md` | Approve or choose a split alternative. |
| 2 | Requirements are measurable and traceable | PASS | `docs/product/REQUIREMENTS.md` | REQ-CR158-001..007 ready for CP2 review. |
| 3 | Scenarios include positive, negative, boundary and precheck coverage | PASS | `docs/product/SCENARIOS.yaml` | SC-CR158-P/N/B/A rows present. |
| 4 | Test matrix preserves no-runtime validation mode | PASS | `docs/product/TEST-MATRIX.md` | Real feed/training/runtime tests authorized count is 0. |
| 5 | MVP scope and out-of-scope boundaries are explicit | PASS | `docs/product/MVP-SCOPE.md` | Runtime and publish remain out of scope. |
| 6 | Next route does not bypass CP3/CP5 | PASS | `process/checks/CP0-CR158.route-plan.json` | CP3/CP5 human gates apply. |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| User approves or requests changes to Decision Brief | PASS | this checkpoint | User replied `approve`; accepted all recommended CP2 decisions. |
| No unresolved blocker remains | PASS | CP2 result | Automatic precheck has no blockers. |
| Next phase is solution-design only after approval | PASS | state update after review | Host-orchestrator will update state to `solution-design`; implementation remains blocked until CP5. |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| CP2 context capsule | `process/context/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-CONTEXT.yaml` | ready | Default read entry. |
| CP1 result | `process/checks/CP1-CR158-EVENT-ML-STRATEGY-ADAPTER-USE-CASE-COMPLETENESS.result.json` | PASS | Product baseline completeness. |
| CP2 result | `process/checks/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-SCOPE.result.json` | PASS | Auto precheck. |
| CP2 manual checkpoint | `process/checkpoints/CP2-CR158-EVENT-ML-STRATEGY-ADAPTER-SCOPE.md` | approved | User accepted the recommended CP2 decisions. |
| Product baseline docs | `docs/product/*` | draft | Updated with CR158 scope. |

## 人工审查结果

| 字段 | 内容 |
|---|---|
| 结论 | approved |
| 审查人 | user |
| 审查时间 | 2026-07-05T16:55:00+08:00 |
| 用户回复 | approve |
| 接受的决策 ID | `DQ-CP2-CR158-UNIFIED-SCOPE`, `DQ-CP2-CR158-NO-RUNTIME`, `DQ-CP2-CR158-GATE-SEQUENCE` |
| 要求修改 | N/A |
| 备注 | CP2 approve only authorizes progression to `solution-design`; it does not authorize runtime, real data access, implementation before CP5, publish, deployment or trading operations. |
