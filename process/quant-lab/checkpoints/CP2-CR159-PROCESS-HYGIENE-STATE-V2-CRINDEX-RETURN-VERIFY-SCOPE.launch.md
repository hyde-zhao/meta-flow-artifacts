# CP2 人工门禁发起：CR-159 Process Hygiene Scope Baseline（修订 v0.2）

## 门禁路径

- **checklist**：`process/checkpoints/CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE.md`
- **自动预检结果**：`process/checks/CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE.result.json`（decision: PASS，无 blocker）
- **context capsule**：`process/context/CP2-CR159-PROCESS-HYGIENE-STATE-V2-CRINDEX-RETURN-VERIFY-SCOPE-CONTEXT.yaml`（read_profile=compact，含 correction_mapping）

## 修订背景

CP2 初版评审指出 5 项问题（cr check 真实失败面 187 ERROR 远超范围、CP6/7 blanket WAIVED 不适合、STATE 0 WARN 与推荐方案冲突、缺 mapping table、cr-tracking 表述）。本修订全部修正：CP8 改 scoped、子项 B 拆 B1/B2、子项 A 扩大范围达真 0 WARN、CP6/7 改条件路由、加 mapping table、cr-tracking 表述拆分。

## 自动预检结论

CP0 `PASS` → CP1 `PASS` → CP2 auto precheck `PASS`。冲突预检：active formal CRs: CR-159 / blocked formal CRs: none（注：cr-tracking 命令整体退出码 1，因历史 invalid lifecycle_status 和 follow-up missing warnings——属本 CR 子项 B2 deferred 范围，不阻塞 CP2）。

## Context Capsule 摘要

- **A. STATE v2 slim**（v0.3：保留运行态契约字段）：`state check` 9 WARN；`slim --dry-run` 7.9KB→5.2KB 归档 source_refs + workflow_health；删 stale phase；保留 current_agent/last_action/cr_tracking_ref/cr_ledger_ref（state-router/SKILL.md:104,114,290 契约），4 residual WARN documented + FU-CR159-002 重访。
- **B1. CR-INDEX scoped**：4 authz L2（CR-151/152/153/154 required_evidence `real_lake_validation`→`static_fixture_only_validation`）+ CR-158 status `closed-current-delivery`→`closed-current-delivery-ready-with-risk`。
- **B2. deferred FU-CR159-001**：182 ERROR（63 cr_ledger invalid status + 58 cr_index invalid status + 57 missing summary_ref + 4 invalid cr_type），44 种历史自定义 status 值，逐 CR 语义判断，不在本 CR 修。
- **C. return/verify 探测**：`story return-check`/`evidence-check`/`verify-packet` 校验深度待探测；CP6/CP7 条件路由。
- **路标回写**：CP8 阶段 v1.2 → v1.3（append-only）。
- **mapping table**：见 capsule `## correction_mapping`（B1 每项 old_value/new_value/reason/backup_ref + STATE field 处置 + B2 分类计数）。
- **CP8 scoped verifier**：见 capsule `cp8_scoped_verifier`（B1 zero + B2 registered + A budget resolved + 4 residual WARN documented；不要求全局 0）。

## 审批者摘要

| 字段 | 内容 |
|---|---|
| 本次确认服务的整体目标 | 确认 CR-159 compact process hygiene 范围（修订 v0.2）：A 达真 0 WARN + B1 scoped 数据修正 + C 探测；B2 deferred FU-CR159-001；CP8 后回写路标 v1.3 |
| 推荐动作 | `approve`：批准三子项范围、7 项决策推荐方案、CP8 scoped 与不授权边界 |
| approve 后会发生什么 | 进入实施：STATE slim + 删 stale phase + 保留 4 运行态字段（residual WARN documented）、CR-INDEX B1 数据修正（备份 + mapping table）、story 工具探测；条件 CP6/CP7 或 N/A；CP8 scoped closure（scoped verifier）+ 路标 v1.3 |
| approve 不授权什么 | Git remote write/push、true release/publish、runtime、真实数据 lake/NAS/provider、凭据、broker、trading、catalog/store/registry 写入、外部框架运行，**重开已关闭的 CR-151/152/153/154/158**，**以及全局 `cr check` 0 ERROR 承诺**（仅 B1 scoped 0 ERROR） |
| 不确认会阻塞什么 | CR-159 实施、CP8、路标回写；STATE 9 WARN / CR-INDEX 187 ERROR / cr-tracking WARNING 保持现状 |

## 决策分层

| 分类 | 数量 |
|---|---:|
| 必须用户决策 | 7 |
| 高风险策略确认 | 1（`DQ-CP2-CR159-NO-REMOTE-RELEASE-RUNTIME`） |
| agent 默认处理 | 4（B2 台账机械分类 + 子项 C N/A 判定） |
| 仅审计记录 | 3 |

## 决策收集覆盖摘要

| 来源 | 候选数 | 纳入数 |
|---|---:|---:|
| formal_cr | 7 | 7 |
| context_capsule | 3 | 3 |
| cp2_review_findings | 5 | 5 |
| tool_evidence | 3 | 3 |
| product_baseline_docs | 0 | 0（N/A） |

## 本轮待人工决策项：7 项

| 决策 ID | 类型 | 推荐方案（简） |
|---|---|---|
| DQ-CP2-CR159-SCOPE | scope | 批准三子项 + B2 deferred + 路标回写；CP8 scoped |
| DQ-CP2-CR159-STATE-UNKNOWN-FIELDS | implementation | slim + 删 stale phase；保留 4 运行态字段（state-router 契约）4 residual WARN documented + FU-CR159-002 重访；非全局 0 WARN |
| DQ-CP2-CR159-AUTHZ-L2-SEMANTICS | implementation | CR-151/152/153/154 required_evidence: `real_lake_validation`→`static_fixture_only_validation`；evidence 标签修正，非能力变更，不重开 closed CR |
| DQ-CP2-CR159-CR158-STATUS | implementation | `closed-current-delivery`→`closed-current-delivery-ready-with-risk`（对齐 CR-156） |
| DQ-CP2-CR159-CP8-SCOPED-CHECK | implementation | CP8 scoped：B1 0 ERROR + B2 deferred；不承诺全局 0 ERROR |
| DQ-CP2-CR159-CP6-CP7-CONDITIONAL | implementation | 条件路由：无代码变更 CP6 N/A/CP7 由 CP8 统一验证；需工具改动则轻量 CP6/CP7 + 回归 |
| DQ-CP2-CR159-NO-REMOTE-RELEASE-RUNTIME | security | 确认不授权边界 + 不承诺全局 0 ERROR + 不重开 closed CR |

完整决策表（备选方案、优劣分析、影响/风险、回退条件）：见 checklist `## 待人工决策清单`。

**blocking / high-risk 决策**：`DQ-CP2-CR159-NO-REMOTE-RELEASE-RUNTIME`（security）、`DQ-CP2-CR159-AUTHZ-L2-SEMANTICS`（涉及 closed CR evidence 修正）、`DQ-CP2-CR159-STATE-UNKNOWN-FIELDS`（删除字段需 grep 验证）。

## 回复与不授权项

如果你回复 approve，表示接受全部 7 项推荐方案；回复 修改: <具体修改点>（如 修改: DQ-CP2-CR159-CR158-STATUS=closed）调整单项；回复 reject 拒绝，CR-159 回退为 candidate。

approve 不表示授权以下不授权项：Git remote write / push、true release execution / publish、runtime、真实数据 lake/NAS/provider、凭据读取、broker、trading、catalog/store/registry 写入、外部框架运行，重开已关闭的 CR-151/152/153/154/158，以及全局 `cr check` 0 ERROR 承诺（仅 B1 scoped 0 ERROR）。
