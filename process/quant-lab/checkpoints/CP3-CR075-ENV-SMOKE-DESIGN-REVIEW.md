---
checkpoint_id: "CP3-CR075-ENV-SMOKE-DESIGN-REVIEW"
checkpoint_name: "CR075 Env Smoke Design Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-16T23:31:26+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-16T23:50:58+08:00"
auto_check_result: "process/checks/CP3-CR075-ENV-SMOKE-DESIGN-CONSISTENCY.md"
context_capsule: "process/context/CP3-CR075-ENV-SMOKE-DESIGN-CONTEXT.yaml"
target:
  phase: "solution-design"
  story_id: ""
  artifacts:
    - "process/changes/CR-075-TARGET-ENV-REBUILD-AND-SMOKE-2026-06-16.md"
---

# CP3 CR075 Env Smoke Design Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR075-ENV-SMOKE-DESIGN-CONSISTENCY.md` | PASS | 0 | 推荐 env rebuild + 最小 smoke 方案；失败路径和备选清楚。 |

## Decision Brief

推荐决策：批准基础依赖同步 + 最小 import smoke 方案。批准后仍不执行，直到 CP5 readiness 也 approved。

备选方案：只做 no-op / dry-run；扩大到 optional dependency groups；扩大到 full tests。推荐方案优先，因为它验证 target 最小可运行性，同时不触碰 data/reports、凭据、provider/lake/catalog 或交易 runtime。

影响维度：`.venv` 写入、依赖下载、最小 smoke 命令、失败处理和后续 runtime readiness 标记。

风险与回退：dependency sync 可能失败或留下 partial `.venv`；本 CR 不自动删除 `.venv`。需要 cleanup 时另起或修改授权。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR075-ENV-SMOKE-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | HLD/ADR N/A：本轮为本机 target env gate，不改代码架构。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 创建 CR075 前无当前 gate DQ；本文件创建后回写。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |
| 自动预检结果 | `process/checks/CP3-CR075-ENV-SMOKE-DESIGN-CONSISTENCY.md` | scanned | 5 | 5 | 方案、备选、失败路径和边界纳入 DQ。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 无新增 Architecture Gray Areas；advisor table 在本文件处理。 |
| 下游正式产物 | CR075 formal CR / pyproject.toml | scanned | 5 | 5 | dependency group 和 smoke 范围纳入 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | “启动CR075”不等于批准具体 smoke 方案。 |

### Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| 基础依赖 + 最小 import smoke | 验证 target 最小可运行；范围清楚 | 会写 `.venv` 并下载依赖 | env / package import | 推荐 | 用户接受本地 env rebuild |
| no-op / dry-run only | 不写运行环境 | 不能证明 target 可运行 | governance only | 备选 | 用户拒绝写 `.venv` |
| optional groups / full tests | 覆盖更广 | 风险、耗时和外部边界更大 | dependency / tests | 不推荐 | 另行授权后切换 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR075-01 | architecture | 是否采用基础依赖 + 最小 smoke 方案？ | 采用。 | no-op；optional groups；full tests。 | 推荐方案可证明最小可运行；备选要么不足要么过宽。 | 写入 `.venv`，运行本地 smoke。 | 需要扩大时重提 CP3/CP5。 |
| DQ-CP3-CR075-02 | implementation | dependency sync 命令是否固定为基础 `uv sync --python 3.11`？ | 固定基础 sync。 | 加 `--group dev` 到 sync；安装所有 groups。 | 推荐方案最小；pytest 可由 `uv run --group dev` 临时保证。 | 避免安装非必要 provider/ML/notebook 依赖。 | smoke 缺 pytest 时由 CP5 方案处理。 |
| DQ-CP3-CR075-03 | implementation | smoke 子集是否只包含 import 和 CR061 package layout？ | 是。 | full pytest；章节研究 smoke。 | 推荐方案快且不碰数据；备选可能触碰 reports/data 或耗时大。 | target runtime readiness 只证明最小层。 | 需要更强验证另起 CR 或修改。 |
| DQ-CP3-CR075-04 | security | 是否禁止 `.env`、reports/data、provider/lake/catalog/runtime？ | 禁止。 | 允许读取 `.env` 或 data；允许 provider fetch。 | 推荐方案避免敏感和外部副作用。 | 不能证明数据链路可用。 | CR076 或专门 runtime CR 处理。 |
| DQ-CP3-CR075-05 | risk_acceptance | 失败后是否自动 cleanup / 重试？ | 不自动，记录 BLOCKED。 | 删除 `.venv` 后重试；切 full reinstall。 | 推荐方案避免破坏；备选需额外授权。 | 可能留下 partial `.venv`。 | 用户明确 cleanup 授权后处理。 |

### 用户需决策事项 summary

| 决策 ID | 用户需决策事项 |
|---|---|
| DQ-CP3-CR075-01 | 是否采用基础依赖 + 最小 smoke 方案。 |
| DQ-CP3-CR075-02 | 是否确认基础 sync 命令边界。 |
| DQ-CP3-CR075-03 | 是否确认 smoke 子集。 |
| DQ-CP3-CR075-04 | 是否确认安全禁止项。 |
| DQ-CP3-CR075-05 | 是否确认失败后不自动 cleanup / 重试。 |

### 不授权项

如果你回复 approve，表示接受本文件 5 项推荐方案；不表示授权以下操作：

- 不授权读取 `.env`、token、密码、cookie、session、private key、账户或交易事实。
- 不授权读取、复制、恢复、比对或删除 `reports/`、`data/`。
- 不授权 provider fetch、lake write、catalog publish 或 MARKET_DATA_LAKE_ROOT replacement。
- 不授权远端 Git 写入、branch/default branch governance、history rewrite、force push、tag、remote deletion 或 set-url。
- 不授权 QMT / MiniQMT runtime、连接、账户查询、下单、撤单、simulation/live。
- 不授权 optional groups、full tests、章节研究 smoke 或 `.venv` cleanup。

### 自动终验授权

- 是否启用：false
- 授权范围：不适用
- 自动通过条件：N/A
- 授权原文：N/A

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 自动预检 PASS | approved | `process/checks/CP3-CR075-ENV-SMOKE-DESIGN-CONSISTENCY.md` |  |
| 设计 context ready | approved | `process/context/CP3-CR075-ENV-SMOKE-DESIGN-CONTEXT.yaml` |  |
| dependency groups 已识别 | approved | `pyproject.toml` |  |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否采用推荐方案 | approved | DQ-CP3-CR075-01 |  |
| 2 | sync 命令是否最小 | approved | DQ-CP3-CR075-02 |  |
| 3 | smoke 子集是否最小 | approved | DQ-CP3-CR075-03 |  |
| 4 | 安全禁止项是否明确 | approved | DQ-CP3-CR075-04 |  |
| 5 | 失败后是否不自动 cleanup | approved | DQ-CP3-CR075-05 |  |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 当前对话 |  |
| CP3 approve 前不执行 env rebuild | approved | 无执行 evidence |  |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP3 auto precheck | `process/checks/CP3-CR075-ENV-SMOKE-DESIGN-CONSISTENCY.md` | approved |  |
| CP3 context | `process/context/CP3-CR075-ENV-SMOKE-DESIGN-CONTEXT.yaml` | approved |  |
| CP3 manual checkpoint | `process/checkpoints/CP3-CR075-ENV-SMOKE-DESIGN-REVIEW.md` | approved |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-16T23:50:58+08:00
- 修改意见：无；用户回复“同意”，视为接受本门禁 Decision Brief 推荐方案。
- 风险接受项：接受 CR075 三门推荐方案；仅授权基础 `uv sync --python 3.11`、import smoke 和 CR061 package layout 小回归；继续不授权 `.env`、reports/data、provider/lake/catalog、远端 Git、QMT/MiniQMT runtime、CR046 recovery、optional groups、full tests 或 destructive cleanup。
