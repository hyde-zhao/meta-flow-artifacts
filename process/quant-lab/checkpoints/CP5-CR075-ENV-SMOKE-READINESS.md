---
checkpoint_id: "CP5-CR075-ENV-SMOKE-READINESS"
checkpoint_name: "CR075 Env Smoke Readiness"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-16T23:31:26+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-16T23:50:58+08:00"
auto_check_result: "process/checks/CP5-CR075-ENV-SMOKE-READINESS.md"
context_capsule: "process/context/CP5-CR075-ENV-SMOKE-READINESS-CONTEXT.yaml"
target:
  phase: "story-planning"
  story_id: ""
  artifacts:
    - "process/changes/CR-075-TARGET-ENV-REBUILD-AND-SMOKE-2026-06-16.md"
---

# CP5 CR075 Env Smoke Readiness 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP5-CR075-ENV-SMOKE-READINESS.md` | PASS | 0 | env rebuild 和 smoke readiness 条件已冻结；实际执行尚未开始。 |

## Decision Brief

推荐决策：批准 CR075 CP5 readiness。若 CP2/CP3/CP5 均 approved，Host Orchestrator 可在执行前复核 CR tracking 和不授权边界，然后执行基础 `uv sync`、import smoke 和 CR061 package layout 小回归。

备选方案：只保留 gate；只做 import smoke 不跑 pytest；扩大到 full tests。推荐方案优先，因为它能证明 target 最小可运行，同时仍避免数据、凭据和外部接口副作用。

影响维度：target `.venv`、基础依赖同步、最小测试证据、STATE / CR-INDEX runtime readiness 标记。

风险与回退：任一命令失败则记录 BLOCKED，不自动 cleanup、删除 `.venv`、扩大 optional groups 或触碰 data/reports。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP5-CR075-ENV-SMOKE-READINESS-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | Story LLD N/A：无代码实现，使用 env smoke readiness gate。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 创建 CR075 前无当前 gate DQ；本文件创建后回写。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |
| 自动预检结果 | `process/checks/CP5-CR075-ENV-SMOKE-READINESS.md` | scanned | 5 | 5 | 执行授权、preflight、smoke、失败路径纳入 DQ。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 无 LLD clarification queue。 |
| 下游正式产物 | CR075 formal CR / smoke test file | scanned | 5 | 5 | command set 和 verification boundary 纳入 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | “启动CR075”不等于执行授权；执行需 CP5 approve。 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP5-CR075-01 | implementation | CP5 approved 后是否允许执行 CR075 env rebuild？ | 允许，但必须三门 approved 且 preflight PASS。 | 只保留 gate；只做人工说明。 | 推荐方案完成 target runtime readiness 验证。 | 写入 `.venv` 并下载依赖。 | preflight FAIL 时阻断。 |
| DQ-CP5-CR075-02 | implementation | 执行前 preflight 如何处理？ | 重跑 CR tracking，并确认不授权项未被扩大。 | 直接执行。 | 推荐方案可审计；直接执行风险高。 | 防止状态分叉和越权。 | 任一失败转 BLOCKED。 |
| DQ-CP5-CR075-03 | implementation | smoke 命令是否固定为三条最小命令？ | 固定为 `uv sync`、import smoke、CR061 package layout pytest。 | 只 import；full tests。 | 推荐方案平衡覆盖和风险。 | 只证明最小 target runtime。 | 需要更广覆盖另行授权。 |
| DQ-CP5-CR075-04 | security | 执行时是否仍禁止 `.env`、reports/data、provider/lake/catalog/runtime/remote？ | 继续禁止。 | 合并 CR076/CR078 或 runtime。 | 推荐方案边界清楚；备选扩大风险。 | 不证明数据链路或交易链路。 | 后续 CR 单独处理。 |
| DQ-CP5-CR075-05 | risk_acceptance | partial `.venv` 或 smoke failure 是否自动清理？ | 不自动清理，记录并等待授权。 | 自动删除 `.venv`；自动重试。 | 推荐方案避免破坏；备选需 destructive 授权。 | 可能留下 partial env。 | 用户授权 cleanup 后处理。 |

### 用户需决策事项 summary

| 决策 ID | 用户需决策事项 |
|---|---|
| DQ-CP5-CR075-01 | 是否允许 CP5 approved 后执行 env rebuild。 |
| DQ-CP5-CR075-02 | 是否确认执行前 preflight。 |
| DQ-CP5-CR075-03 | 是否确认三条最小 smoke 命令。 |
| DQ-CP5-CR075-04 | 是否确认执行时仍禁止敏感和外部动作。 |
| DQ-CP5-CR075-05 | 是否确认 partial env / smoke failure 不自动清理。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权读取、复制、恢复、比对或删除 `reports/`、`data/`。
- 不授权 provider fetch、lake write、catalog publish 或 MARKET_DATA_LAKE_ROOT replacement。
- 不授权远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 QMT / MiniQMT runtime、连接、账户查询、下单、撤单、simulation/live。
- 不授权 optional groups、full tests、章节研究 smoke、删除 `.venv` 或 destructive cleanup。
- 不授权恢复或推进 CR046。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP5 自动预检 PASS | approved | `process/checks/CP5-CR075-ENV-SMOKE-READINESS.md` |  |
| CR075 readiness context ready | approved | `process/context/CP5-CR075-ENV-SMOKE-READINESS-CONTEXT.yaml` |  |
| smoke 子集存在 | approved | `tests/test_cr061_package_import_layout.py` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否允许 approved 后执行 env rebuild | approved | DQ-CP5-CR075-01 |  |
| 2 | preflight 是否严格 | approved | DQ-CP5-CR075-02 |  |
| 3 | smoke 命令是否固定 | approved | DQ-CP5-CR075-03 |  |
| 4 | 敏感和外部动作是否继续禁止 | approved | DQ-CP5-CR075-04 |  |
| 5 | partial env 是否不自动清理 | approved | DQ-CP5-CR075-05 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话 |  |
| CP5 approve 前不执行 env rebuild | approved | 无执行 evidence |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP5 auto precheck | `process/checks/CP5-CR075-ENV-SMOKE-READINESS.md` | approved |  |
| CP5 context | `process/context/CP5-CR075-ENV-SMOKE-READINESS-CONTEXT.yaml` | approved |  |
| CP5 manual checkpoint | `process/checkpoints/CP5-CR075-ENV-SMOKE-READINESS.md` | approved |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-16T23:50:58+08:00
- 修改意见：无；用户回复“同意”，视为接受本门禁 Decision Brief 推荐方案。
- 风险接受项：接受 CR075 三门推荐方案；仅授权基础 `uv sync --python 3.11`、import smoke 和 CR061 package layout 小回归；继续不授权 `.env`、reports/data、provider/lake/catalog、远端 Git、QMT/MiniQMT runtime、CR046 recovery、optional groups、full tests 或 destructive cleanup。
