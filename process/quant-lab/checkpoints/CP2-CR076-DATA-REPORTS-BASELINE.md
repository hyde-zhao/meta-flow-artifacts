---
checkpoint_id: "CP2-CR076-DATA-REPORTS-BASELINE"
checkpoint_name: "CR076 Data/reports Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-17T00:18:01+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-17T00:41:44+08:00"
auto_check_result: "process/checks/CP2-CR076-DATA-REPORTS-BASELINE.md"
context_capsule: "process/context/CP2-CR076-DATA-REPORTS-CONTEXT.yaml"
auto_final_authorization: false
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/changes/CR-076-DATA-REPORTS-ACCESS-POLICY-2026-06-17.md"
---

# CP2 CR076 Data/reports Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR076-DATA-REPORTS-BASELINE.md` | PASS | 0 | CR076 formal CR 已创建；本轮只确认 data/reports access policy 范围，不做真实访问。 |

## Decision Brief

推荐决策：批准 CR076 CP2 范围基线，正式启动 data/reports access policy 门禁；本批准只确认策略范围和后续授权边界，不授权读取、列出、复制、恢复、比对或删除 `reports/`、`data/`。

备选方案：保持 `CR076-candidate` 不启动；扩大为真实 data/reports 访问 CR；取消 CR076。推荐方案优先，因为它先把敏感数据边界和后续恢复路径说清楚，避免把未授权的真实访问混入本轮。

影响维度：CR tracking、STATE、reports/data 访问边界、NAS retained archive、local retained assets、provider rebuild 后续路径、CR077/CR078/CR046 分流。

风险与回退：若范围表述不清，回退为 `CR076-candidate`；若用户需要真实访问，需要另起 execution / runtime authorization CR。回退只修改 process ledger，不做 data/reports cleanup。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR076-DATA-REPORTS-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | SGQ N/A：用户明确要求 CR075 完成后继续 CR076；CR076 来源于已登记 follow-up candidate。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 创建 CR076 前无当前 gate DQ；本文件创建后回写。 |
| CR076 formal CR | `process/changes/CR-076-DATA-REPORTS-ACCESS-POLICY-2026-06-17.md` | scanned | 5 | 5 | 范围、冲突、授权和不授权边界纳入 DQ。 |
| CR075 CP8 approval | `process/checkpoints/CP8-CR075-DELIVERY-READINESS.md` | scanned | 1 | 1 | 用户要求 CR075 完成后继续 CR076，作为启动依据。 |
| CR-INDEX follow-up | `process/changes/CR-INDEX.yaml` | scanned | 4 | 3 | CR076/CR077/CR078 分流纳入；CR026 与本轮无关。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 本轮不是新增产品场景发现；范围来自 CR follow-up。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR076-01 | scope | 是否正式启动 CR076？ | 启动 CR076，进入 CP3/CP5 策略门禁。 | 保持 candidate；取消 CR076。 | 推荐方案响应用户“继续处理076”；保持 candidate 会保留 data/reports 策略空白。 | 新增正式 CR 和待审门禁。 | reject 时恢复为 candidate 或取消。 |
| DQ-CP2-CR076-02 | scope | 范围是否限定为 data/reports access policy，不做真实访问？ | 限定为策略、边界、候选路径和后续门禁。 | 立即纳入真实读取；立即纳入恢复 / 删除。 | 推荐方案可审计且低触碰；备选会进入敏感数据操作。 | target data/reports readiness 仍未知。 | 需要真实访问时另起 execution CR。 |
| DQ-CP2-CR076-03 | security | 是否继续禁止 `.env`、凭据、账户和交易事实？ | 继续禁止。 | 同时授权凭据读取；授权账户 / 交易事实查询。 | 推荐方案维持最小权限；备选需更高安全审批。 | 防止凭据和交易事实泄露。 | 用户另起 credential / runtime CR 时切换。 |
| DQ-CP2-CR076-04 | architecture | 是否把 local retained、NAS archive、provider rebuild 作为后续候选路径而非本轮执行？ | 三条路径均只作为候选，不在本轮访问。 | 立即选定并执行其中一条；取消候选路径。 | 推荐方案先定策略；立即执行会扩大权限面。 | 后续恢复路径暂未验证。 | CP3/CP5 后另起真实访问 CR。 |
| DQ-CP2-CR076-05 | risk_acceptance | 是否接受 CR076 通过前 target data/reports readiness 仍未知？ | 接受未知状态，仅记录为后续风险。 | 现在验证 readiness；取消 data/reports 后续处理。 | 推荐方案不触碰敏感路径；现在验证需要真实访问授权。 | 数据可用性和完整性仍未证明。 | 用户要求真实性验证时另起 CR。 |

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
| CP2 自动预检 PASS | approved | `process/checks/CP2-CR076-DATA-REPORTS-BASELINE.md` |  |
| CR076 formal CR 存在 | approved | `process/changes/CR-076-DATA-REPORTS-ACCESS-POLICY-2026-06-17.md` |  |
| context ready | approved | `process/context/CP2-CR076-DATA-REPORTS-CONTEXT.yaml` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否正式启动 CR076 | approved | DQ-CP2-CR076-01 | 用户回复“同意”，接受推荐方案。 |
| 2 | 范围是否限定为 policy-only | approved | DQ-CP2-CR076-02 | 用户回复“同意”，接受推荐方案。 |
| 3 | 凭据和交易事实是否继续禁止 | approved | DQ-CP2-CR076-03 | 用户回复“同意”，接受推荐方案。 |
| 4 | 后续路径是否只作为候选 | approved | DQ-CP2-CR076-04 | 用户回复“同意”，接受推荐方案。 |
| 5 | readiness 未知风险是否接受 | approved | DQ-CP2-CR076-05 | 用户回复“同意”，接受推荐方案。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话 | 用户回复“同意”，按 approve 处理。 |
| CP2 approve 后仍不执行真实访问 | approved | 不授权项 | approve 仅授权 policy-only 门禁。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 auto precheck | `process/checks/CP2-CR076-DATA-REPORTS-BASELINE.md` | approved |  |
| CP2 context | `process/context/CP2-CR076-DATA-REPORTS-CONTEXT.yaml` | approved |  |
| CP2 manual checkpoint | `process/checkpoints/CP2-CR076-DATA-REPORTS-BASELINE.md` | approved |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-17T00:41:44+08:00
- 修改意见：无；用户回复“同意”，接受 DQ-CP2-CR076-01..05 推荐方案。
- 风险接受项：接受 CR076 CP2 policy-only 范围和 data/reports readiness 未知风险；确认 CP2 approve 不授权读取、列出、复制、恢复、比对或删除 `reports/`、`data/`，不授权 `.env` / 凭据 / provider / lake / catalog / 远端 Git / runtime / CR046 / cleanup。
