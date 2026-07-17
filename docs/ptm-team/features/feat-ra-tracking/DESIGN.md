---
status: draft
version: "1.0"
source_cr: "CR-030"
source_hld: "docs/design/HLD.md (v1.2, REV-03)"
feature_id: "FEAT-RA-TRACKING"
agent: "ptm-tse"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-TRACKING: 闭环跟踪与有效性 — Feature 设计

> 对应 HLD REV-03 的 `improvement-tracker` Skill 跟踪侧 + 人工 reviewer 边界。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-se | 初始 Feature 设计。 |
| 1.1 | 2026-07-16 | host-orchestrator | CP5 Round 4：MeasureBaseline 补齐 `proposed_status` / `refresh_hint`，明确系统仅写提示、reviewer 唯一可改正式状态。 |

## 1. Feature 概述

### 目标

让质量负责人基于已批准的 CA/PA，跟踪行动项执行、检查有效性、审查观察窗和复发指标，做出可解释的关闭/保持开放决策。同时支持 S2 措施基线的刷新提示。

### 成功标准

| 指标 | 度量方式 | 目标值 |
|---|---|---|
| 关闭条件不可绕过 | 任一条件不足时 RA 保持开放 | fixture 证明 |
| 不等同完成率 | 关闭基于有效性而非行动项完成 | 审查证据 |
| 措施刷新安全 | 不自动改变批准状态或下游任务 | fixture 证明 |

## 2. 关键对象

### 2.1 Action Item

```yaml
action_id: string
proposal_ref: string         # 关联 CA/PA Proposal
owner: string
due_date: date
status: enum                 # 'not-started' | 'in-progress' | 'done' | 'overdue'
blockers: string[]
implementation_evidence: string
```

### 2.2 Effectiveness Check

```yaml
check_id: string
action_ref: string
method: string               # 验证方式
window_days: int             # 观察窗口
result: enum                 # 'planned' | 'passed' | 'failed' | 'inconclusive'
recurrence_measure:
  same_category_count: int
  total_observation_count: int
  recurrence_rate: float
notes: string
```

### 2.3 Closure Decision

```yaml
ra_id: string
decision: enum               # 'closed' | 'open'
conditions:
  all_actions_complete: boolean
  effectiveness_passed: boolean
  observation_window_satisfied: boolean
  no_same_category_recurrence: boolean
residual_risks: string[]
follow_up_actions: string[]
reviewer: string
decided_at: datetime
```

### 2.4 MeasureBaseline

```yaml
baseline_id: string
proposal_ref: string
version: int
scope: string
approval_ref: string
implementation_evidence: string
effectiveness_evidence: string
observation_window:
  start: date
  end: date
status: enum                 # 'needs-baseline' | 'active' | 'completed' | 'needs-review' | 'superseded'
proposed_status: enum | null # 系统建议状态；仅提示，不改变正式 status
refresh_hint: string         # 系统生成的刷新原因
refreshed_at: datetime
refreshed_by: string
```

## 3. 状态机

### Action Item 状态
```
not-started → in-progress → done
                │              │
                └── overdue ◄──┘ (过期未完成)
```

### 关闭条件
```
所有 action items done?
    AND effectiveness check passed?
    AND observation window satisfied?
    AND no same-category recurrence?
        │
   Yes  └──► RA closed
   No   └──► RA open + residual risks + follow-up actions
```

## 4. S2 措施刷新规则

| 触发 | 措施状态变化 | 提示 |
|---|---|---|
| 新增相关问题单 | needs-review | "新问题可能影响措施有效性" |
| 措施实施完成 | completed | "可进入观察期" |
| 观察窗通过 | active（持续） | "观察窗已满足" |
| 同类问题复发 | needs-review | "同类复发，措施可能失效" |
| 无基线 | needs-baseline | "需先建立措施基线" |

**禁止自动操作**：不自动改变批准状态、不自动关闭、不自动修改下游任务。

## 5. Gotchas

- **不要把行动项完成率当成关闭依据**：必须检查有效性和复发
- **不要自动关闭 RA**：只有人工 reviewer 能做关闭决定
- **不要把 30 天观察窗伪装成真实观察**：fixture/dry-run 只能验证规则，真实有效性需人工业务证据
- **不要让措施刷新自动失效或关闭**：只产出提示，不改变状态
- **不要在没有 baselines 时判断措施失效**：标为 needs-baseline

---

# FEAT-RA-TRACKING — 测试计划

| ID | 场景 | Story | 预期 |
|---|---|---|---|
| T-TRK-01 | 行动项状态流转 | ST-RA-04 | not-started→in-progress→done |
| T-TRK-02 | 过期行动项标记 | ST-RA-04 | 超过 due_date → overdue |
| T-TRK-03 | 所有行动项完成 + 有效性通过 + 观察窗满足 + 无复发 → 关闭 | ST-RA-04 | 关闭条件全部满足才可关闭 |
| T-TRK-04 | 任一条件不足 → 保持开放 | ST-RA-04 | 输出 residual_risks + follow_up |
| T-TRK-05 | 有效性检查结果（passed/failed/inconclusive） | ST-RA-04 | 检查结果正确反映在 closure |
| T-TRK-06 | MeasureBaseline 创建与管理 | ST-RA-06.3 | baseline 含版本/范围/审批引用 |
| T-TRK-07 | 无基线 → needs-baseline | ST-RA-06.3 | 不判措施失效，提示建立基线 |
| T-TRK-08 | 措施刷新提示（保持/完成/需复核/失效） | ST-RA-06.3 | 只产出提示，不自动改变状态 |
| T-TRK-09 | 不自动关闭 RA | ST-RA-04 | fixture 证明无自动关闭路径 |
| T-TRK-10 | 不自动改变批准状态 | ST-RA-06.3 | 措施刷新不改变 approval_status |

---

# FEAT-RA-TRACKING — 任务清单

### ST-RA-04: 闭环跟踪与有效性决策

**TASK-TRK-01**: 在 `skills/improvement-tracker/SKILL.md` 追加跟踪侧定义
**TASK-TRK-02**: 实现 Action Item 状态机（not-started→in-progress→done/overdue）
**TASK-TRK-03**: 实现 Effectiveness Check 模型（planned/passed/failed/inconclusive）
**TASK-TRK-04**: 实现 Closure Decision 模型（四个关闭条件）
**TASK-TRK-05**: 实现关闭条件不可绕过逻辑
**TASK-TRK-06**: 创建 Action Item / Effectiveness Check / Closure Decision 模板

### ST-RA-06.3-TRACK: 措施基线管理与刷新提示

**TASK-TRK-07**: 实现 MeasureBaseline 模型与管理
**TASK-TRK-08**: 实现措施刷新规则（保持/完成/需复核/失效 + needs-baseline）
**TASK-TRK-09**: 实现不自动改变批准状态的门控
**TASK-TRK-10**: 创建 Fixture（各种措施刷新场景）
