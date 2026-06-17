---
checkpoint_id: "CP3-CR074-CUTOVER-DESIGN-REVIEW"
checkpoint_name: "CR074 Cutover Design Review"
type: "auto_then_manual"
status: "approved"
owner: "host-orchestrator"
created_at: "2026-06-16T22:14:44+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-16T22:41:34+08:00"
auto_check_result: "process/checks/CP3-CR074-CUTOVER-DESIGN-CONSISTENCY.md"
context_capsule: "process/context/CP3-CR074-CUTOVER-DESIGN-CONTEXT.yaml"
target:
  phase: "solution-design"
  story_id: ""
  artifacts:
    - "process/changes/CR-074-ROOT-CUTOVER-DAILY-WORKFLOW-SWITCH-2026-06-16.md"
---

# CP3 CR074 Cutover Design Review 人工审查

## 自动预检摘要

| 预检文件 | 结论 | 阻断项 | 说明 |
|---|---|---:|---|
| `process/checks/CP3-CR074-CUTOVER-DESIGN-CONSISTENCY.md` | PASS | 0 | 推荐 logical cutover marker，回退和不授权边界明确。 |

## Decision Brief

推荐决策：批准 logical cutover marker 方案。批准后只把 target 标记为后续 authoritative daily workflow root，并保留旧根为 legacy root；不改 shell / IDE / cron / service，不做 symlink 或物理目录移动。

备选方案：只发人工操作说明；系统级 symlink / shell 配置切换；继续停留旧根。推荐方案优先，因为它有状态证据、可回退且不引入系统级副作用。

影响维度：后续会话入口、状态文档、root authority 语义；不影响代码运行、data lake、远端 Git 或 runtime。

风险与回退：target 尚未 env/data/runtime ready；旧根保留为回退路径。若需要系统级配置修改，另起 CR。

### Context Capsule Summary

| 字段 | 内容 |
|---|---|
| capsule 路径 | `process/context/CP3-CR074-CUTOVER-DESIGN-CONTEXT.yaml` |
| capsule 状态 | ready |
| read_profile | compact |
| 默认读取策略 | 先读 capsule；仅在缺失、冲突、字段不足、人工审计或深度评审时读取完整文档 |
| 全文档读取扩展 | 0 次 |
| 缺失 / waived 理由 | HLD/ADR N/A：本轮为本机根语义治理，不改代码架构。 |

### Decision Collection Coverage

| 来源 | 路径 / 对象 | 扫描状态 | 候选问题数 | 纳入待决策数 | 分类 / N/A 原因 |
|---|---|---:|---:|---:|---|
| STATE pending queue | `process/STATE.md.human_gate_decisions.pending_human_decisions[]` | scanned | 0 | 0 | 创建 CR074 前无当前 gate DQ；本文件创建后回写。 |
| 委托 Agent 交还摘要 | `process/handoffs/*RETURN-SUMMARY.md` | n/a | 0 | 0 | 本轮不调度功能 agent。 |
| 自动预检结果 | `process/checks/CP3-CR074-CUTOVER-DESIGN-CONSISTENCY.md` | scanned | 5 | 5 | 方案、备选、回退、不授权边界纳入 DQ。 |
| discussion log / checkpoint | `process/discussions/*` / `process/checks/*DISCUSSION-CHECKPOINT.json` | n/a | 0 | 0 | 无新增 Architecture Gray Areas；advisor table 在 CR074 处理。 |
| 下游正式产物 | CR074 formal CR | scanned | 5 | 5 | 设计方案和后续分流纳入 DQ。 |
| 用户显式选择题 | 当前对话 | scanned | 1 | 1 | “继续”只授权启动 gate，不授权实际 cutover。 |

### Advisor Table

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| Logical cutover marker | 有状态证据；可回退；无系统级副作用 | 需要用户/后续会话遵守新 root | STATE / CR-INDEX / handoff | 推荐 | target readiness 已通过且旧根保留 |
| Manual instruction only | 零写入 | 容易产生隐式状态分叉 | 用户操作习惯 | 不推荐 | 用户拒绝状态文件变更时 |
| Symlink / shell config | 体验透明 | 高风险，可能影响其他工具 | OS / shell / IDE / service | 不推荐 | 另起 ops CR 并明确授权 |
| Stay on old root | 风险最低 | target 无法成为日常入口 | 工作流入口 | 备选 | 用户 reject CR074 |

### 待人工决策清单

| 决策 ID | 决策类型 | 待确认问题 | 推荐方案 | 备选方案 | 优劣分析 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|---|---|---|
| DQ-CP3-CR074-01 | architecture | 是否采用 logical cutover marker 方案？ | 采用。 | manual instruction only；symlink/shell config；stay old root。 | 推荐方案可审计且可回退；系统级方案风险高。 | 后续 root authority 语义变化。 | 需要系统级切换时另起 CR。 |
| DQ-CP3-CR074-02 | implementation | cutover 写入面是否只限 STATE / CR-INDEX / CR074 evidence？ | 是。 | 同步更多 docs/process；修改 shell/IDE。 | 推荐方案最小；备选过宽。 | 减少分叉和副作用。 | 需要扩大写入面时重提 CP3/CP5。 |
| DQ-CP3-CR074-03 | security | 是否禁止 symlink、目录移动和旧根 cleanup？ | 禁止。 | 允许 symlink；允许目录移动。 | 推荐方案无破坏动作；备选需 destructive 授权。 | 旧根回退路径保留。 | old root retirement 走 CR077。 |
| DQ-CP3-CR074-04 | risk_acceptance | 是否接受 cutover 后 target 仍未 env/data/runtime ready？ | 接受。 | 先 CR075/CR076，再 CR074。 | 推荐方案先完成入口治理；备选更保守但延后。 | 后续运行需单独验证。 | 用户要求运行保障时先 CR075/CR076。 |
| DQ-CP3-CR074-05 | follow_up_tracking | 后续 CR075-CR078 如何处理？ | 保持候选，不自动启动。 | 立即创建；取消。 | 推荐方案聚焦；立即创建会开多门禁；取消会丢路线。 | 后续治理可追溯。 | 用户明确启动某项时转正式 CR。 |

### 用户需决策事项 summary

| 决策 ID | 用户需决策事项 |
|---|---|
| DQ-CP3-CR074-01 | 是否采用 logical cutover marker 方案。 |
| DQ-CP3-CR074-02 | 是否确认写入面只限 STATE / CR-INDEX / CR074 evidence。 |
| DQ-CP3-CR074-03 | 是否确认禁止 symlink、目录移动和旧根 cleanup。 |
| DQ-CP3-CR074-04 | 是否接受 target 尚未 env/data/runtime ready。 |
| DQ-CP3-CR074-05 | 是否保留 CR075-CR078 为候选。 |

## Entry Criteria

| 条目 | 状态 | 证据 | 审查意见 |
|---|---|---|---|
| CP3 自动预检 PASS | approved | `process/checks/CP3-CR074-CUTOVER-DESIGN-CONSISTENCY.md` | 用户回复“同意”，接受 CP3 推荐方案。 |
| 设计 context ready | approved | `process/context/CP3-CR074-CUTOVER-DESIGN-CONTEXT.yaml` | context ready。 |
| Advisor table 完成 | approved | 本文件 | logical marker 方案获批准。 |

## Checklist

| # | 检查项 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|---|
| 1 | 是否采用 logical marker | approved | DQ-CP3-CR074-01 | 接受推荐方案。 |
| 2 | 写入面是否最小 | approved | DQ-CP3-CR074-02 | 接受推荐方案。 |
| 3 | 是否禁止 destructive cutover | approved | DQ-CP3-CR074-03 | 接受推荐方案。 |
| 4 | runtime-ready 风险是否接受 | approved | DQ-CP3-CR074-04 | 接受推荐方案。 |
| 5 | follow-up 是否保留候选 | approved | DQ-CP3-CR074-05 | 接受推荐方案。 |

## Exit Criteria

| 条目 | 审查结果 | 证据 | 审查意见 |
|---|---|---|---|
| 用户明确 approve / 修改 / reject | approved | 用户回复“同意” | 按 `approve` 处理。 |
| CP3 approve 前不执行 cutover | approved | 无 CP3 前 cutover evidence | CP3 前未执行。 |

## Deliverables

| 交付物 | 路径 | 审查结果 | 审查意见 |
|---|---|---|---|
| CP3 auto precheck | `process/checks/CP3-CR074-CUTOVER-DESIGN-CONSISTENCY.md` | approved |  |
| CP3 context | `process/context/CP3-CR074-CUTOVER-DESIGN-CONTEXT.yaml` | approved |  |
| CP3 manual checkpoint | `process/checkpoints/CP3-CR074-CUTOVER-DESIGN-REVIEW.md` | approved |  |

## 人工审查结果

- 结论：`approved`
- 审查人：user
- 审查时间：2026-06-16T22:41:34+08:00
- 修改意见：无；用户回复“同意”，接受 DQ-CP3-CR074-01..05 推荐方案。
- 风险接受项：接受 logical marker 方案；不授权 symlink、目录移动、旧根 cleanup、shell / IDE / cron / service 修改。
