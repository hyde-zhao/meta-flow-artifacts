---
checkpoint_id: "CP3-CR076-DATA-REPORTS-POLICY-REVIEW"
checkpoint_name: "CR076 Data/reports Policy Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T00:18:01+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T00:41:44+08:00"
auto_check_result: "process/checks/CP3-CR076-DATA-REPORTS-POLICY-CONSISTENCY.md"
context_capsule: "process/context/CP3-CR076-DATA-REPORTS-POLICY-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "solution-design"
  story_id: ""
  artifacts:
    - "process/changes/CR-076-DATA-REPORTS-ACCESS-POLICY-2026-06-17.md"
---

# CP3 CR076 Data/reports Policy Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR076-DATA-REPORTS-POLICY-CONSISTENCY.md` | PASS | 0 | policy-first / no-data-touch 方案可审；本轮不做真实访问。 |

## Decision Brief

推荐决策：采用 policy-first / no-data-touch 方案，先确认 data/reports 访问策略、证据要求、脱敏边界和后续授权条件；真实读取、复制、恢复、比对、删除或 provider rebuild 必须另起执行 CR。

备选方案：直接选择 local retained assets；直接选择 NAS archive；直接选择 provider rebuild；取消 data/reports 后续处理。推荐方案优先，因为它在信息不足且权限敏感时保持可审计边界。

影响维度：访问架构、恢复路径决策、运行授权、证据脱敏、memory policy、后续 rollback / cleanup 边界。

风险与回退：若策略不被接受，回退为 CR076 pending 或 candidate，不修改数据；若未来需要执行某条恢复路径，切换到新的 runtime_authorization / execution CR。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR076-DATA-REPORTS-POLICY-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| Architecture Gray Areas | N/A：CR076 是访问策略门禁，不改代码架构或平台路径。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| CP3 Context Capsule | `process/context/CP3-CR076-DATA-REPORTS-POLICY-CONTEXT.yaml` | scanned | 5 | 5 | 策略、路径、授权、脱敏、回退纳入 DQ。 |
| CR076 formal CR | `process/changes/CR-076-DATA-REPORTS-ACCESS-POLICY-2026-06-17.md` | scanned | 5 | 5 | 推荐策略和不授权边界纳入。 |
| 自动预检结果 | `process/checks/CP3-CR076-DATA-REPORTS-POLICY-CONSISTENCY.md` | scanned | 5 | 5 | policy consistency 已纳入。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 无新增架构灰区交互；本轮以 CR follow-up 约束为准。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR076-01 | architecture | 是否采用 policy-first / no-data-touch 方案？ | 采用；先写策略和授权边界，不访问数据。 | 直接进入真实访问；取消 CR076。 | 推荐方案权限最小；直接访问会触碰敏感路径。 | data/reports 事实仍未知。 | 需要真实访问时另起执行 CR。 |
| DQ-CP3-CR076-02 | architecture | 后续恢复路径如何建模？ | local retained、NAS archive、provider rebuild 三者并列为候选路径。 | 仅保留 local；仅保留 NAS；仅保留 provider rebuild。 | 推荐方案保留可选性；单一路径可能过早排除备选。 | 后续计划需要再决策。 | CP5 或后续 CR 选择具体路径。 |
| DQ-CP3-CR076-03 | runtime_authorization | 后续真实访问是否必须 per-run explicit runtime authorization？ | 必须逐 run 明确授权。 | 一次性授权后续所有访问。 | 推荐方案风险可控；一次性授权边界过宽。 | 降低误读和越权风险。 | 用户明确批准具体执行 CR 时切换。 |
| DQ-CP3-CR076-04 | security | 样本 / 清单证据是否必须脱敏且不写入 memory？ | 必须脱敏；不把样本、清单、凭据或未脱敏日志写入 memory。 | 允许保存完整清单；允许保存样本。 | 推荐方案符合 memory policy；备选有泄露风险。 | 限制后续证据表达方式。 | 经用户和安全门禁明确批准后才可放宽。 |
| DQ-CP3-CR076-05 | implementation | 失败 / 回退是否仅回滚 policy ledger，不做 cleanup？ | 是，仅回滚 process ledger。 | 失败时删除或清理 data/reports；失败时恢复旧根。 | 推荐方案不触碰数据；备选是破坏性或高风险动作。 | 保留本地和归档状态不变。 | 需要 cleanup 时另起 destructive CR。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权读取、列出、复制、恢复、比对或删除 `reports/`、`data/`。
- 不授权读取、打印、记录、复制或迁移 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权 provider fetch、lake write、catalog publish 或 MARKET_DATA_LAKE_ROOT replacement。
- 不授权远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 QMT / MiniQMT runtime、连接、账户查询、下单、撤单、simulation/live。
- 不授权删除、重命名或移动旧根 `/home/hyde/workspace/local_backtest`。
- 不授权恢复或推进 CR046。
- 不授权 optional dependency groups、full tests、删除 `.venv` 或 destructive cleanup。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 自动预检 PASS | approved | `process/checks/CP3-CR076-DATA-REPORTS-POLICY-CONSISTENCY.md` |  |
| CP3 context ready | approved | `process/context/CP3-CR076-DATA-REPORTS-POLICY-CONTEXT.yaml` |  |
| 推荐策略已记录 | approved | CR076 formal CR §推荐策略 |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否采用 policy-first / no-data-touch | approved | DQ-CP3-CR076-01 | 用户回复“同意”，接受推荐方案。 |
| 2 | 三条后续路径是否并列候选 | approved | DQ-CP3-CR076-02 | 用户回复“同意”，接受推荐方案。 |
| 3 | 后续真实访问是否逐 run 授权 | approved | DQ-CP3-CR076-03 | 用户回复“同意”，接受推荐方案。 |
| 4 | 证据脱敏和 memory 边界是否接受 | approved | DQ-CP3-CR076-04 | 用户回复“同意”，接受推荐方案。 |
| 5 | 回退是否仅限 policy ledger | approved | DQ-CP3-CR076-05 | 用户回复“同意”，接受推荐方案。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话 | 用户回复“同意”，按 approve 处理。 |
| CP3 approve 后仍不执行真实访问 | approved | 不授权项 | approve 仅授权 policy-first / no-data-touch 方案。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP3 auto precheck | `process/checks/CP3-CR076-DATA-REPORTS-POLICY-CONSISTENCY.md` | approved |  |
| CP3 context | `process/context/CP3-CR076-DATA-REPORTS-POLICY-CONTEXT.yaml` | approved |  |
| CP3 manual checkpoint | `process/checkpoints/CP3-CR076-DATA-REPORTS-POLICY-REVIEW.md` | approved |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T00:41:44+08:00
- 修改意见：无；用户回复“同意”，接受 DQ-CP3-CR076-01..05 推荐方案。
- 风险接受项：接受 policy-first / no-data-touch 方案；确认后续真实访问必须逐 run 明确授权，样本 / 清单证据必须脱敏且不写入 memory，失败 / 回退只回滚 policy ledger。
