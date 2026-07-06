---
checkpoint_id: "CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE"
checkpoint_name: "CR159 Process Hygiene Scope Baseline (revised v0.2)"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-07-06T22:05:54+08:00"
revised_at: "2026-07-06T22:40:00+08:00"
reviewed_by: "user"
reviewed_at: "2026-07-06T23:10:00+08:00"
auto_check_result: "process/checks/CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE.result.json"
target:
  phase: "requirement-clarification"
  cr_id: "CR-159"
  artifacts:
    - "process/changes/CR-159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-2026-07-06.md"
    - "process/context/CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE-CONTEXT.yaml"
    - "process/checks/CP1-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE-COMPLETENESS.result.json"
    - "process/checks/CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE.result.json"
revision_notes: "v0.3 per 2nd CP2 review: subitem A keeps 4 runtime-contract fields (current_agent/last_action/cr_tracking_ref/cr_ledger_ref) as residual WARN + FU-CR159-002 revisit; only deletes stale phase + archives source_refs/workflow_health; added CP8 scoped verifier. v0.2: CP8 scoped, B1/B2 split, CP6/CP7 conditional, mapping table, 7 decisions, cr-tracking split."
---

# CP2 CR159 Process Hygiene Scope Baseline 人工审查（修订 v0.2）

## 修订背景

CP2 初版评审（用户 findings）指出 5 项问题：(1) 真实 `cr check` 失败面 187 ERROR 远超子项 B 范围；(2) CP6/CP7 blanket WAIVED 不适合子项 C 可能的工具实现；(3) STATE slim 退出标准与推荐方案冲突（0 WARN vs 残余 WARN）；(4) agent 默认处理项缺 mapping table；(5) cr-tracking 表述需拆分。本修订按 findings 全部修正。

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CR159 formal CR exists | PASS | `process/changes/CR-159-...-2026-07-06.md` | 长命名正式 CR（v0.2 修订）。 |
| CP0 request intake passed | PASS | `process/checks/CP0-CR159-REQUEST-INTAKE.result.json` | CP0 decision PASS。 |
| CP1 scope completeness passed | PASS | `process/checks/CP1-CR159-...-SCOPE-COMPLETENESS.result.json` | CP1 decision PASS。 |
| Context capsule ready | PASS | `process/context/CP2-CR159-...-SCOPE-CONTEXT.yaml` | read_profile=compact，v0.2 修订。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | CR159 三子项范围 + B2 deferred 是否明确 | PASS | `DQ-CP2-CR159-SCOPE` | 待用户确认。 |
| 2 | STATE 6 unknown field + source_refs 处置是否达真 0 WARN | PASS | `DQ-CP2-CR159-STATE-UNKNOWN-FIELDS` | 待用户确认。 |
| 3 | CR-151/152/153/154 authz L2 是否按 evidence 标签修正 | PASS | `DQ-CP2-CR159-AUTHZ-L2-SEMANTICS` | 待用户确认。 |
| 4 | CR-158 status 修正值 | PASS | `DQ-CP2-CR159-CR158-STATUS` | 待用户确认。 |
| 5 | CP8 验收是 scoped 还是全局 0 ERROR | PASS | `DQ-CP2-CR159-CP8-SCOPED-CHECK` | 待用户确认。 |
| 6 | CP6/CP7 是 blanket WAIVED 还是条件路由 | PASS | `DQ-CP2-CR159-CP6-CP7-CONDITIONAL` | 待用户确认。 |
| 7 | 不授权边界 + 不承诺全局 0 ERROR + 不重开 closed CR | PASS | `DQ-CP2-CR159-NO-REMOTE-RELEASE-RUNTIME` | 待用户确认。 |
| 8 | 是否明确不修改产品基线 | PASS | `product_baseline.refresh_required=false` | 路标回写为过程文档。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP2 auto precheck PASS | PASS | `process/checks/CP2-CR159-...-SCOPE.result.json` | 无 blocker。 |
| Decisions approved | PENDING | 下方人工审查结果 | 7 项决策待用户评审。 |
| Unauthorized scope explicit | PASS | 下方审批者摘要 / 决策表 | approve 不授权真实运行、外部写入、重开 closed CR、不承诺全局 0 ERROR。 |
| Next route clear | PASS | `next_route_if_approved=implementation_then_CP8` | CP2 approval 后实施；条件 CP6/CP7 或 N/A；CP8 scoped closure。 |
| CP8 scoped verifier defined | PASS | context capsule `cp8_scoped_verifier` | B1 zero + B2 registered + A budget resolved + 4 residual WARN documented；不要求全局 0。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| CR159 formal CR | `process/changes/CR-159-...-2026-07-06.md` | PASS | v0.2 修订。 |
| CR159 summary | `process/changes/summaries/CR-159.summary.json` | PASS | 轻量入口。 |
| CP1 result | `process/checks/CP1-CR159-...-SCOPE-COMPLETENESS.result.json` | PASS | scope completeness。 |
| CP2 context capsule | `process/context/CP2-CR159-...-SCOPE-CONTEXT.yaml` | PASS | v0.2，含 correction_mapping。 |
| CP2 result JSON | `process/checks/CP2-CR159-...-SCOPE.result.json` | PASS | 自动预检。 |
| CP2 human checkpoint | `process/checkpoints/CP2-CR159-...-SCOPE.md` | pending | 待用户评审回填。 |

## Decision Brief

### 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-159 compact process hygiene 范围（修订 v0.2）：A) STATE v2 slim 达真 0 WARN + B1) CR-INDEX scoped 数据修正（authz L2 + CR-158）+ C) return/verify 探测；B2 全局 legacy deferred FU-CR159-001；CP8 后回写路标 v1.3。 |
| 推荐动作 | `approve`：批准三子项范围、7 项决策推荐方案、CP8 scoped 与不授权边界。 |
| approve 后会发生什么 | 进入实施：STATE slim + 删 stale phase + 保留 4 运行态字段（residual WARN documented）、CR-INDEX B1 数据修正（备份 + mapping table）、story 工具探测；条件 CP6/CP7 或 N/A；CP8 scoped closure（scoped verifier）+ 路标 v1.3。 |
| approve 不授权什么 | Git remote write / push、true release execution / publish、runtime、真实数据 lake/NAS/provider、凭据读取、broker、trading、catalog/store/registry 写入、外部框架运行，**重开已关闭的 CR-151/152/153/154/158**，**以及全局 `cr check` 0 ERROR 承诺**（仅 B1 scoped 0 ERROR）。 |
| 不确认会阻塞什么 | 不确认会阻塞 CR-159 实施、CP8、路标回写；STATE 9 WARN、CR-INDEX 187 ERROR、cr-tracking WARNING 保持现状。 |

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule | `process/context/CP2-CR159-...-SCOPE-CONTEXT.yaml` |
| read_profile | `compact` |
| 默认读取策略 | capsule-first；默认消费 CR summary、CP0/CP1/CP2 result、路标 §7 ref、meta-flow 工具核验证据和 correction_mapping。 |
| 全文档读取 | 本次无需读取完整产品需求 / HLD / LLD；三子项缺口由 `meta-flow state check` / `cr check` / `cr-tracking` / `state slim --dry-run` 只读工具机械核验。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---|---:|---:|---|
| formal_cr | `process/changes/CR-159-...-2026-07-06.md` | scanned | 7 | 7 | Scope、STATE fields、authz L2、CR-158 status、CP8 scoped、CP6/7 conditional、security。 |
| context_capsule | `process/context/CP0-CR159.context.json` | scanned | 3 | 3 | 用户指定三项决策映射。 |
| cp2_review_findings | 用户评审 findings 2026-07-06 | scanned | 5 | 5 | Findings 1-5 驱动 CP8 scoped、CP6/7 conditional、STATE 0 WARN 对齐、mapping table、cr-tracking 拆分。 |
| tool_evidence | `meta-flow state check / cr check / cr-tracking / state slim --dry-run` | scanned | 3 | 3 | 工具核验缺口喂入 STATE/authz/CR158 决策。 |
| product_baseline_docs | USE-CASES / REQUIREMENTS / SCENARIOS / MVP | N/A | 0 | 0 | 本 CR 不改变产品基线。 |

### 决策分层

| 分类 | 数量 | 处理方式 |
|---|---:|---|
| 必须用户决策 | 7 | 已列入待决策清单，待用户评审。 |
| 高风险策略确认 | 1 | `DQ-CP2-CR159-NO-REMOTE-RELEASE-RUNTIME` 明确不授权边界 + 不承诺全局 0 ERROR + 不重开 closed CR。 |
| agent 默认处理 | 4 | 25 missing summary_ref 补齐（B2 deferred）、3 invalid lifecycle_status 映射（B2 deferred）、6 follow-up missing from items 补登（B2 deferred）、子项 C 探测 N/A 判定。 |
| 仅审计记录 | 3 | CR-156 已关闭 FU-CR154-001/FU-CR152-001 事实、路标 v1.2 滞后、CR-158 readiness_status 已是 ready_with_risk。 |

注：B2 全部缺口（63 cr_ledger invalid status + 58 cr_index invalid status + 57 missing summary_ref + 4 invalid cr_type）转 FU-CR159-001 candidate，不占决策项；agent 默认处理仅在 B2 台账登记时机械分类计数。

### 用户需决策事项

- `DQ-CP2-CR159-SCOPE`
- `DQ-CP2-CR159-STATE-UNKNOWN-FIELDS`
- `DQ-CP2-CR159-AUTHZ-L2-SEMANTICS`
- `DQ-CP2-CR159-CR158-STATUS`
- `DQ-CP2-CR159-CP8-SCOPED-CHECK`
- `DQ-CP2-CR159-CP6-CP7-CONDITIONAL`
- `DQ-CP2-CR159-NO-REMOTE-RELEASE-RUNTIME`

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR159-SCOPE | scope | 是否批准三子项 + B2 deferred FU-CR159-001 + CP8 路标 v1.3 回写？ | 批准三子项 + B2 deferred + 路标回写；CP8 scoped。 | A. 扩大 B 到全局 187 ERROR；B. 只做 A，B/C deferred。 | 推荐 scoped 可执行；A 需逐 CR 语义判断；B 遗留 authz L2 + CR-158。 | 影响实施和 CP8 范围。 | 若用户要求全局 0 ERROR 或扩大到 push/release/runtime，回退 requirement-clarification。 |
| DQ-CP2-CR159-STATE-UNKNOWN-FIELDS | implementation | STATE 6 unknown field + source_refs 超 budget 如何处置？（state-router 契约依赖 4 字段） | slim 归档 source_refs + workflow_health + 删 stale phase；保留 current_agent/last_action/cr_tracking_ref/cr_ledger_ref（state-router/SKILL.md:104,114,290 契约），4 residual WARN documented + FU-CR159-002 重访。 | A. 扩展 v2 schema allowlist（转 FU-CR159-002）；B. 删 4 字段 + 改 state-router/TEMPLATE（破坏契约）。 | 推荐不破坏运行态契约；A 超范围；B 删除会被 state-router 补回。 | 4 residual WARN 保留，非 0 WARN；需 CP8 显式记录（R-CR159-STATE-RESIDUAL-WARN）。 | 若用户坚持真 0 WARN，转 FU-CR159-002 schema 扩展或另起 CR 改 state-router 契约。 |
| DQ-CP2-CR159-AUTHZ-L2-SEMANTICS | implementation | CR-151/152/153/154 required_evidence 含 real_lake_validation 与 NO_REAL_LAKE_READ_OR_WRITE 冲突如何修正？ | required_evidence: real_lake_validation → static_fixture_only_validation；evidence 标签修正，非能力变更，不重开 closed CR；备份 + mapping table。 | A. 保留标 evidence_status=deferred；B. 移除不替换。 | 推荐与实际验证模式一致；A 保留矛盾；B 损失追溯。 | 修正 closed CR evidence 可能被误读为能力变更（R-CR159-AUTHZ-L2-DATA-FIX-VS-CAPABILITY-CHANGE）。 | 若用户判定为能力变更，回退并另起正式 CR 重开 CR-151/152/153/154。 |
| DQ-CP2-CR159-CR158-STATUS | implementation | CR-158 status=closed-current-delivery 无效，修正为什么值？ | closed-current-delivery → closed-current-delivery-ready-with-risk（对齐 CR-156）。 | A. closed；B. 保留转 B2 deferred。 | 推荐与 CR-156 一致；A 丢失语义；B 不修。 | 纯数据修正。 | 无实质回退；可恢复原值。 |
| DQ-CP2-CR159-CP8-SCOPED-CHECK | implementation | CP8 验收是全局 cr check 0 ERROR 还是 scoped？ | CP8 scoped：B1（authz L2 + CR-158 + CR-159 item）0 ERROR；B2 转 FU-CR159-001；不承诺全局 0 ERROR。 | A. 全局 0 ERROR（扩大 B 到 187）；B. CP8 不做 cr check 验收。 | 推荐 scoped 可执行诚实；A 工作量大；B 丢失 cr 治理验收。 | 全局 cr check 仍有 B2 ERROR，可能被误读为未完成（R-CR159-CP8-SCOPED-NOT-GLOBAL）。 | 若用户要求全局 0 ERROR，扩大 B2 进本 CR 或另起 CR。 |
| DQ-CP2-CR159-CP6-CP7-CONDITIONAL | implementation | CP6/CP7 是 blanket WAIVED 还是条件路由？ | 条件路由：子项 C 无代码变更 → CP6 N/A、CP7 由 CP8 统一验证；需增强工具 → 轻量 CP6/CP7 evidence + 回归。 | A. blanket WAIVED；B. 一律完整 CP6/CP7。 | 推荐匹配子项 C 不确定性；A 可能漏工具验证；B 过重。 | 子项 C 探测结果未知（R-CR159-CP6-CP7-CONDITIONAL）。 | 若子项 C 需工具改动，自动切换到轻量 CP6/CP7 路由。 |
| DQ-CP2-CR159-NO-REMOTE-RELEASE-RUNTIME | security | 确认不授权 Git remote write / true release / publish / runtime / real data / credential / NAS/provider / broker / trading / catalog-store-registry write / external framework / 重开 closed CR / 全局 cr check 0 ERROR 承诺？ | 确认不授权边界 + 不承诺全局 0 ERROR；后续只允许本地 hygiene + 路标 append-only 回写。 | A. 允许后续单独 commit preparation gate；B. 延后全部到 release-readiness CR。 | 推荐风险最低；A 减少等待但需新门禁；B 审计更重。 | 防止 hygiene/CR-INDEX 修正被误读为 push/publish/runtime/重开 closed CR/全局 0 ERROR。 | 任一动作需外部写入/真实运行/凭据/真实数据时立即阻断并另起 gate。 |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-07-06T23:10:00+08:00
- 修改意见：接受全部 7 项推荐方案；实施时 cp8_scoped_verifier 已改为真实可执行 shell（`test "$(...)" -eq 0`，v0.3 修复）。
- 风险接受项：`R-CR159-STATE-RESIDUAL-WARN`（4 residual WARN 保留，state-router 契约）、`R-CR159-CP8-SCOPED-NOT-GLOBAL`（不承诺全局 0 ERROR）
- 已接受决策项：`DQ-CP2-CR159-SCOPE`、`DQ-CP2-CR159-STATE-UNKNOWN-FIELDS`、`DQ-CP2-CR159-AUTHZ-L2-SEMANTICS`、`DQ-CP2-CR159-CR158-STATUS`、`DQ-CP2-CR159-CP8-SCOPED-CHECK`、`DQ-CP2-CR159-CP6-CP7-CONDITIONAL`、`DQ-CP2-CR159-NO-REMOTE-RELEASE-RUNTIME`
