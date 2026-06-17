---
checkpoint_id: "CP2-CR075-ENV-REBUILD-BASELINE"
checkpoint_name: "CR075 Env Rebuild Baseline"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-16T23:31:26+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-16T23:50:58+08:00"
auto_check_result: "process/checks/CP2-CR075-ENV-REBUILD-BASELINE.md"
context_capsule: "process/context/CP2-CR075-ENV-REBUILD-CONTEXT.yaml"
target:
  phase: "requirement-clarification"
  story_id: ""
  artifacts:
    - "process/changes/CR-075-TARGET-ENV-REBUILD-AND-SMOKE-2026-06-16.md"
---

# CP2 CR075 Env Rebuild Baseline 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP2-CR075-ENV-REBUILD-BASELINE.md` | PASS | 0 | CR075 候选已转正式 CR，范围和不授权边界完整；尚未执行 env rebuild。 |

## Decision Brief

推荐决策：批准 CR075 CP2 范围基线，允许继续进入 CP3/CP5 门禁；实际 `uv sync` 和 smoke 仍需 CP2/CP3/CP5 全部 approved 后才可执行。

备选方案：只保留 CR075 为 formal pending；取消 CR075；扩大到 data/reports 或 optional groups 后重提。推荐方案优先，因为它先冻结最小 target runtime readiness 范围，避免把 data/reports、凭据或交易 runtime 混入本轮。

影响维度：target `.venv` 重建授权边界、基础依赖同步、最小 import / package layout smoke、STATE / CR-INDEX tracking。

风险与回退：dependency sync 可能创建 `.venv` 并下载依赖；CP2 不授权执行。若后续 preflight 或 smoke 失败，记录 BLOCKED，不自动删除 `.venv` 或扩大范围。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP2-CR075-ENV-REBUILD-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | Scenario Gray Areas N/A：用户明确启动 CR075，范围由 CR073/CR074 follow-up 定义。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 创建 CR075 前无当前 gate DQ；本文件创建后回写。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |
| 自动预检结果 | `process/checks/CP2-CR075-ENV-REBUILD-BASELINE.md` | scanned | 5 | 5 | 范围、冲突、授权和不授权边界纳入 DQ。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 本轮不是场景发现阶段；CR follow-up 范围清晰。 |
| 下游正式产物 | CR075 formal CR / CR073 / CR074 follow-up | scanned | 5 | 5 | follow-up 转正式 CR 和边界纳入 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | “启动CR075”只授权启动 formal CR，不等于执行 env rebuild。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP2-CR075-01 | scope | 是否确认正式启动 CR075？ | 启动 CR075，进入 CP3/CP5 门禁。 | 保持候选；取消 CR075。 | 推荐方案响应用户启动请求；保持候选不解决 target runtime readiness。 | 会新增正式 CR 和状态索引。 | 用户 reject 时保持候选或取消。 |
| DQ-CP2-CR075-02 | scope | CR075 是否只覆盖 target env rebuild 和最小 smoke？ | 是，只覆盖基础依赖、import smoke、package layout 小回归。 | 扩大到 data/reports；扩大到 optional groups。 | 推荐方案最小可审计；备选风险更高。 | target 是否可最小运行可被验证。 | 需要数据或 optional groups 时重提。 |
| DQ-CP2-CR075-03 | security | 是否继续禁止 `.env`、reports/data、provider/lake/catalog、remote Git、QMT/MiniQMT？ | 继续禁止。 | 合并 CR076/CR078 或 runtime。 | 推荐方案边界清晰；备选混入敏感面。 | 避免凭据、数据和外部写入风险。 | 需要访问时另起 CR 或修改本 CR。 |
| DQ-CP2-CR075-04 | implementation | CP2 approve 是否授权立即执行 `uv sync`？ | 不授权；需 CP2/CP3/CP5 全部 approved。 | CP2 后立即执行。 | 推荐方案符合门禁；备选越过设计/执行 readiness。 | 避免过早写入 `.venv`。 | 三门 approved 后进入执行。 |
| DQ-CP2-CR075-05 | risk_acceptance | 是否接受 target 当前仍非 runtime-ready？ | 接受；CR075 通过后再判定 runtime readiness。 | 先不启动；直接 full tests。 | 推荐方案分阶段；full tests 风险和耗时高。 | 当前 target 仍可能无法运行。 | smoke 失败时记录 BLOCKED。 |

### 用户需决策事项 summary

| 决策 ID | 用户需决策事项 |
|---|---|
| DQ-CP2-CR075-01 | 是否正式启动 CR075。 |
| DQ-CP2-CR075-02 | 是否确认 CR075 只覆盖 target env rebuild 和最小 smoke。 |
| DQ-CP2-CR075-03 | 是否确认继续禁止敏感和外部动作。 |
| DQ-CP2-CR075-04 | 是否确认 CP2 approve 不等于执行授权。 |
| DQ-CP2-CR075-05 | 是否接受 target 仍非 runtime-ready，等待 CR075 smoke 判定。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权读取、复制、恢复、比对或删除 `reports/`、`data/`。
- 不授权 provider fetch、lake write、catalog publish 或 MARKET_DATA_LAKE_ROOT replacement。
- 不授权远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 QMT / MiniQMT runtime、连接、账户查询、下单、撤单、simulation/live。
- 不授权删除、重命名或移动旧根。
- 不授权恢复或推进 CR046。
- 不授权 CP2 后立即执行 `uv sync` 或 smoke。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP2 自动预检 PASS | approved | `process/checks/CP2-CR075-ENV-REBUILD-BASELINE.md` |  |
| CR075 formal CR 存在 | approved | `process/changes/CR-075-TARGET-ENV-REBUILD-AND-SMOKE-2026-06-16.md` |  |
| context ready | approved | `process/context/CP2-CR075-ENV-REBUILD-CONTEXT.yaml` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否正式启动 CR075 | approved | DQ-CP2-CR075-01 |  |
| 2 | 范围是否最小 | approved | DQ-CP2-CR075-02 |  |
| 3 | 不授权边界是否清楚 | approved | DQ-CP2-CR075-03 |  |
| 4 | CP2 是否不授权执行 | approved | DQ-CP2-CR075-04 |  |
| 5 | target 非 runtime-ready 风险是否接受 | approved | DQ-CP2-CR075-05 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话 |  |
| CP2 approve 前不执行 env rebuild | approved | 无执行 evidence |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP2 auto precheck | `process/checks/CP2-CR075-ENV-REBUILD-BASELINE.md` | approved |  |
| CP2 context | `process/context/CP2-CR075-ENV-REBUILD-CONTEXT.yaml` | approved |  |
| CP2 manual checkpoint | `process/checkpoints/CP2-CR075-ENV-REBUILD-BASELINE.md` | approved |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-16T23:50:58+08:00
- 修改意见：无；用户回复“同意”，视为接受本门禁 Decision Brief 推荐方案。
- 风险接受项：接受 CR075 三门推荐方案；仅授权基础 `uv sync --python 3.11`、import smoke 和 CR061 package layout 小回归；继续不授权 `.env`、reports/data、provider/lake/catalog、远端 Git、QMT/MiniQMT runtime、CR046 recovery、optional groups、full tests 或 destructive cleanup。
