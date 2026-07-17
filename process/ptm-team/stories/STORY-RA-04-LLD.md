---
story_id: "STORY-RA-04"
canonical_story_id: "ST-RA-04"
title: "闭环跟踪与有效性决策"
lld_policy: "full-lld"
tier: "M"
feature: "FEAT-RA-TRACKING"
design_evidence_type: "full-lld"
source_hld: "docs/design/HLD.md (v1.2, REV-03)"
source_feature_design: "docs/features/feat-ra-tracking/DESIGN.md (v1.0)"
depends_on: ["ST-RA-03"]
shared_with: ["ST-RA-03", "ST-RA-06.3-TRACK"]
created_by: "meta-dev"
created_at: "2026-07-16"
---

# ST-RA-04 高层设计（LLD）：闭环跟踪与有效性决策

## 0. 工程依据与模板索引

| 来源 | 消费内容 |
|---|---|
| HLD / Feature DESIGN / Feature Matrix | 闭环四条件、reviewer 门控和 `full-lld` 约束 |
| Story `ST-RA-04` | 输出文件、验收标准和依赖 |

本文保留原有详细章节；目标、需求、模块拆分、代码结构、数据模型、API、流程、技术细节、安全、测试、实施、风险与 DoD 分别由后续编号章节定义。

> 对应 HLD REV-03 的 `improvement-tracker` Skill 跟踪侧 + 人工 reviewer 边界。
> 本章 LLD 证据覆盖 14 个语义要点。

## LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-dev | 初始 LLD：行动项状态机、有效性检查、关闭决策四条件、不可绕过门控 |

---

## 1. Goal

在 improvement-tracker Skill 中追加闭环跟踪侧。基于已批准 CA/PA（ST-RA-03 产出），管理行动项执行状态、检查有效性、审查观察窗和复发指标，做出可解释的关闭/保持开放决策。

**核心约束：** 只有人工 reviewer 能做关闭决定；关闭基于有效性而非完成率；不自动关闭 RA。

**核心成功标准：**

| 指标 | 目标值 | 验证方式 |
|---|---|---|
| 关闭条件不可绕过 | 任一条件不足时 RA 保持开放 | fixture：逐一缺失四个关闭条件，验证输出 open + residual_risks |
| 不应等于完成率 | 行动项全 done 但有效性 failed → 保持开放 | fixture 证明 |
| 不自动关闭 | 系统/Skill 不能触发 closure decision | fixture：模拟自动关闭 → 拒绝 |

---

## 2. Requirements

### 2.1 Functional Requirements

| 需求 ID | 描述 | 本 Story 覆盖 |
|---|---|---|
| REQ-RA-009 | 让质量负责人依据行动、验证和观察证据决定是否关闭，而不是只看完成率 | 完全覆盖：四条件关闭决策 |
| REQ-RA-013 | 度量：分析覆盖、及时性、采纳、复发、按时闭环 | 部分覆盖：复发指标（same_category_count, recurrence_rate） |

### 2.2 Non-Functional Requirements

| 质量属性 | 设计目标 | 实现手段 |
|---|---|---|
| 可审计性 | 每个关闭决策可追溯 | Closure Decision 含所有条件评估结果 + reviewer + decided_at |
| 安全性 | 不自动关闭 | 状态机规则：closure decision 仅由 reviewer 触发 |
| 可靠性 | 条件检查幂等 | 每次检查独立评估四个条件，不依赖历史缓存 |

### 2.3 Scope Boundary

| 负责事项 | 不负责事项 |
|---|---|
| Action Item 状态机（not-started→in-progress→done/overdue） | 行动项的实际执行（由人工 Owner 负责） |
| Effectiveness Check 模型 | 有效性的真实度量（由人工提供证据，fixture 验证规则） |
| Closure Decision 模型（四条件） | 替代 Owner 完成行动 |
| 关闭条件不可绕过逻辑 | 自动关闭、通知下游 |
| 30 天观察窗的规则验证 | 30 天真实观察证据（由人工提供） |

---

## 3. 模块拆分与职责

本 Story 在 `improvement-tracker` Skill 中追加跟踪侧，分为 3 个子模块：

| 模块 | 职责 | 输入 | 输出 | 禁止行为 |
|---|---|---|---|---|
| **Action Item 管理器** | 行动项创建、状态流转、过期检测 | 已批准 CA/PA Proposal | action-item.yaml | 自动完成行动项 |
| **Effectiveness Checker** | 有效性检查创建、结果记录、复发统计 | action items + 观察窗口 | effectiveness-check.yaml | 替代人工判断有效性 |
| **Closure Decision Engine** | 四条件评估、关闭/保持开放输出 | action items + effectiveness checks + observation window + recurrence data | closure-decision.yaml | 自动关闭、绕过条件 |

**此次追加的模板文件：**

```
skills/improvement-tracker/
├── SKILL.md                         # 追加 §4 闭环跟踪
└── templates/
    ├── capa-proposal.yaml           # ST-RA-03 已创建
    ├── approved-input.yaml          # ST-RA-03 已创建
    ├── action-item.yaml             # 本 Story 创建
    ├── effectiveness-check.yaml     # 本 Story 创建
    └── closure-decision.yaml        # 本 Story 创建
```

---

## 4. 代码结构与文件影响范围

### 4.1 文件清单

| 文件 | 操作 | 写入模块 | 说明 |
|---|---|---|---|
| `skills/improvement-tracker/SKILL.md` | 追加（§4 闭环跟踪） | 所有子模块 | 追加行动项、有效性、关闭决策流程 |
| `skills/improvement-tracker/templates/action-item.yaml` | 创建 | Action Item 管理器 | 行动项 schema + 示例 |
| `skills/improvement-tracker/templates/effectiveness-check.yaml` | 创建 | Effectiveness Checker | 有效性检查 schema + 示例 |
| `skills/improvement-tracker/templates/closure-decision.yaml` | 创建 | Closure Decision Engine | 关闭决策 schema + 示例 |

### 4.2 文件所有权

| 文件 | owner | shared_with | write_section |
|---|---|---|---|
| `skills/improvement-tracker/SKILL.md` | FEAT-RA-TRACKING | ST-RA-03, ST-RA-06.3-TRACK | §4 action items, effectiveness, closure |
| `skills/improvement-tracker/templates/action-item.yaml` | FEAT-RA-TRACKING | — | 独占 |
| `skills/improvement-tracker/templates/effectiveness-check.yaml` | FEAT-RA-TRACKING | — | 独占 |
| `skills/improvement-tracker/templates/closure-decision.yaml` | FEAT-RA-TRACKING | — | 独占 |

**串行约束**：ST-RA-03 必须先完成 SKILL.md §3 CA/PA 侧写入；本 Story 在 §4 追加跟踪侧；ST-RA-06.3-TRACK 在 §5 追加 MeasureBaseline 侧。

### 4.3 读取依赖

| 数据对象 | 来源 | 读取方式 | 用途 |
|---|---|---|---|
| Approved Improvement Input | ST-RA-03 产出 | 只读引用 | 关联 action items 到 CA/PA |
| CA/PA Proposal（approved） | ST-RA-03 产出 | 只读引用 | 获取 Owner、due_date、validation_method |
| `measure_link` | SQLite（ST-RA-INGEST-DB 创建） | 只读查询 | 复发统计（相同类别问题单计数） |

### 4.4 不涉及的文件

以下文件不属于本 Story 范围：
- `skills/itr-ticket-ingestion/*`（F-020 专有）
- `skills/reverse-analysis/*`（F-021 专有）
- `data/schema.sql`、`data/dao.py`（SQLite，只读查询 measure_link；写入由 F-020 负责）

---

## 5. 数据模型与持久化设计

### 5.1 Action Item Schema

```yaml
# action-item.yaml — 本 Story 创建
action_id: string           # 唯一标识，格式：ACTION-{proposal_id}-{seq}
proposal_ref: string        # 关联 CA/PA Proposal ID
input_ref: string           # 关联 Approved Improvement Input ID
owner: string               # 执行 Owner
due_date: date              # 到期日期
status: enum                # 'not-started' | 'in-progress' | 'done' | 'overdue'
description: string         # 行动描述
blockers: string[]          # 阻塞项列表
implementation_evidence: string  # 实施证据（文件引用或描述）
created_at: datetime
updated_at: datetime
```

**约束：**
- `action_id` 全局唯一
- `status` 受状态机规则控制
- `blockers` 为空数组时缺省为 `[]`
- `overdue` 状态由系统基于 `due_date` 自动标记（当 `status ∈ {not-started, in-progress}` 且 `now > due_date` 时）

### 5.2 Effectiveness Check Schema

```yaml
# effectiveness-check.yaml — 本 Story 创建
check_id: string            # 唯一标识，格式：EFF-CHECK-{action_id}-{seq}
action_ref: string          # 关联 Action Item ID
method: string              # 验证方式（如 "回归测试"、"人工审查"）
window_days: int            # 观察窗口天数（默认 30）
result: enum                # 'planned' | 'passed' | 'failed' | 'inconclusive'
recurrence_measure:
  same_category_count: int      # 观察窗内同类问题数
  total_observation_count: int  # 观察窗内总问题数
  recurrence_rate: float        # 复发率
notes: string               # reviewer 备注
checked_by: string          # 检查人
checked_at: datetime
```

**约束：**
- `check_id` 全局唯一
- `result` 初始值为 `planned`
- `window_days` 默认 30，可由 reviewer 调整
- `recurrence_rate` 计算规则：`same_category_count / total_observation_count`（当 total>0 时），否则 N/A
- `recurrence_measure` 数据来源：SQLite `measure_link` 表查询（同 proposal_ref 的问题单）

### 5.3 Closure Decision Schema

```yaml
# closure-decision.yaml — 本 Story 创建
ra_id: string               # 关联 RA Report ID
proposal_id: string         # 关联 CA/PA Proposal ID
decision: enum              # 'open' | 'closed'
conditions:
  all_actions_complete: boolean         # 条件 1：所有行动项完成
  effectiveness_passed: boolean         # 条件 2：有效性检查通过
  observation_window_satisfied: boolean # 条件 3：观察窗满足
  no_same_category_recurrence: boolean  # 条件 4：无同类复发
residual_risks: string[]    # 剩余风险
follow_up_actions: string[] # 后续行动
reviewer: string            # 关闭决定人
decided_at: datetime
```

**约束：**
- `decision = "closed"` 时，四个 `conditions` 必须全部为 `true`
- 任一条件为 `false` → `decision = "open"` + `residual_risks` 和 `follow_up_actions` 必填
- `reviewer` 和 `decided_at` 必填
- 关闭决策仅由人工 reviewer 触发，系统不得自动执行

### 5.4 持久化说明

本 Story 的数据对象以文件化 YAML 形式持久化。与 ST-RA-03 一致，不依赖 SQLite 写入。复发统计（`same_category_count`）通过只读查询 SQLite `measure_link` 表获取，但不在本 Story 中写入该表。

---

## 6. API / Interface 设计

### 6.1 improvement-tracker Skill（跟踪侧）接口

**触发条件：**
- Skill 名称：`improvement-tracker`
- 触发词：`improvement-tracker`、`改进跟踪`、`行动项`、`有效性`、`关闭决策`
- 前提：已有已批准的 CA/PA Proposal（ST-RA-03 产出）

**执行模式：**

| 模式 | 触发描述 | 输入 | 输出 |
|---|---|---|---|
| `manage-actions` | 管理行动项：创建、状态更新、过期检测 | proposal_id | action-item.yaml |
| `effectiveness-check` | 执行有效性检查 | action_id + observation window data | effectiveness-check.yaml |
| `closure-review` | reviewer 执行关闭决策 | ra_id + proposal_id | closure-decision.yaml |
| `status-overview` | 查看跟踪状态概览 | proposal_id | 行动项列表 + 有效性状态 + 关闭条件摘要 |

### 6.2 上游依赖接口

| 上游对象 | 提供者 | 本 Story 消费方式 | 必须字段 |
|---|---|---|---|
| CA/PA Proposal（approved） | ST-RA-03 产出 | 只读引用 | proposal_id, kind, owner, due_date, validation_method |
| Approved Improvement Input | ST-RA-03 产出 | 只读引用 | input_id, proposal_id, scope, acceptance_criteria |
| `measure_link` 表 | SQLite（F-020） | 只读查询 | proposal_ref, ticket_id, window_start, window_end |

### 6.3 下游消费接口

本 Story 的 Closure Decision 产出是跟踪闭环的终点：
- 人工 reviewer 审查 closure-decision.yaml 做出最终决定
- 若 `decision = "open"`，后续行动项（follow_up_actions）反馈回 Action Item 管理器
- 关闭决策结果可被 ST-RA-06.3-TRACK 的措施刷新逻辑读取

---

## 7. 核心处理流程

### 7.1 Action Item 状态机

```
not-started ──(start)──► in-progress ──(complete)──► done
     │                       │
     │   ┌──(due_date past)──┘
     │   ▼
     └─ overdue              overdue ←(due_date past)─ in-progress
```

**状态迁移规则：**

| 源状态 | 目标状态 | 触发者 | 条件 |
|---|---|---|---|
| `not-started` | `in-progress` | Owner/Reviewer | 手动标记开始 |
| `in-progress` | `done` | Owner/Reviewer | 手动标记完成 |
| `not-started` | `overdue` | 系统自动 | `now > due_date` |
| `in-progress` | `overdue` | 系统自动 | `now > due_date` |
| `overdue` | `done` | Owner/Reviewer | 手动标记完成 |

**自动 overridden 说明：** 系统只自动标记过期，不自动完成、不自动关闭、不自动改变其他状态。

### 7.2 有效性检查流程

```
Action Items (全部 done?)
      │
      ▼
[1] 创建 Effectiveness Check（初始 result = "planned"）
      │
      ▼
[2] 等待观察窗满足（window_days ≥ 30 天 或 reviewer 手动触发）
      │
      ▼
[3] 收集复发数据（SQLite measure_link 查询）
      │ same_category_count, total_observation_count
      │ recurrence_rate = same_category_count / total_observation_count
      │
      ▼
[4] 人工 reviewer 填写 result
      │
      ├── "passed"         → 有效性确认
      ├── "failed"         → 有效性不通过
      └── "inconclusive"   → 证据不足以判断
```

### 7.3 关闭决策四条件评估

```
[1] 收集数据
      │
      ├── 条件 1: all_actions_complete
      │     → 所有 action items status == "done"？
      │
      ├── 条件 2: effectiveness_passed
      │     → 所有 effectiveness checks result == "passed"？
      │
      ├── 条件 3: observation_window_satisfied
      │     → 所有 effectiveness checks 的 window_days 已满足？
      │
      └── 条件 4: no_same_category_recurrence
            → 所有 effectiveness checks 的 recurrence_rate == 0？
      │
      ▼
[2] 全部 true？
      ├── YES → decision = "closed"
      │      → 输出空的 residual_risks 和 follow_up_actions（或填 N/A）
      │
      └── NO  → decision = "open"
            → 输出不满足的条件 + residual_risks + follow_up_actions
      │
      ▼
[3] 人工 reviewer 确认 → 签入 reviewer + decided_at
```

### 7.4 关闭条件不可绕过

```text
门控规则（硬编码，不可由 Skill 或 AI 覆盖）：

if decision == "closed":
    assert conditions.all_actions_complete == True
    assert conditions.effectiveness_passed == True
    assert conditions.observation_window_satisfied == True
    assert conditions.no_same_category_recurrence == True
else:
    # "open" 时 residual_risks 和 follow_up_actions 必须有内容
    assert len(residual_risks) > 0 or len(follow_up_actions) > 0
```

### 7.5 异常路径

| 场景 | 触发条件 | 行为 | 输出 |
|---|---|---|---|
| 行动项全部 done 但有效性 failed | effectiveness result = "failed" | 关闭条件 2 不满足 | decision = "open" + residual_risks |
| 观察窗未满足 | window_days < 30 且 reviewer 未手动豁免 | 关闭条件 3 不满足 | decision = "open" + 提示等待观察窗 |
| 有同类复发 | recurrence_rate > 0 | 关闭条件 4 不满足 | decision = "open" + 复发详情 |
| 无关联 CA/PA Proposal | proposal_ref 指向不存在的 proposal | 拒绝操作 | 提示"关联的 CA/PA 不存在" |
| 自动关闭尝试 | Skill/AI 试图直接设置 decision="closed" | 拒绝 | 提示"关闭决定需人工 reviewer 确认" |

---

## 8. 技术设计细节

### 8.1 过期检测机制

Action Item 的 overdue 状态由系统在每次状态查询时动态计算：

```text
overdue_condition:
  (status == "not-started" OR status == "in-progress") AND now > due_date
```

- 不修改持久化状态文件，仅动态计算
- 在 `status-overview` 输出中标记
- Reviewer 可选择将 overdue 项手动标记为 done

### 8.2 观察窗计算

`observation_window_satisfied` 基于 Effectiveness Check 的 `window_days` 和 `checked_at`：

```text
window_satisfied = (now - checked_at) >= window_days
```

默认 `window_days = 30`。Reviewer 可基于业务判断手动设置 `window_satisfied = true`（如观察期已过但未记录起始时间时）。

### 8.3 复发统计查询

复发数据从 SQLite `measure_link` 表查询（只读）：

```sql
SELECT
  COUNT(*) as same_category_count,
  (SELECT COUNT(*) FROM measure_link WHERE proposal_ref = ?) as total_observation_count
FROM measure_link
WHERE proposal_ref = ?
  AND window_start >= ?
  AND window_end <= ?
```

本 Story 只定义查询接口，SQLite 写入由 F-020 负责。

### 8.4 SKILL.md 追加策略

本 Story 在 SKILL.md 的已有 CA/PA 侧（§3）之后追加 §4 闭环跟踪。结构：

```markdown
## 3. CA/PA 治理（由 ST-RA-03 写入）
...
## 4. 闭环跟踪（由 ST-RA-04 写入）
### 4.1 Action Item 管理
### 4.2 有效性检查
### 4.3 关闭决策
### 4.4 关闭条件不可绕过规则
## 5. 措施基线管理（由 ST-RA-06.3-TRACK 写入）
```

追加时：
1. 先读取已有 SKILL.md 内容
2. 找到 §3 结束位置（下一个 `## ` 级标题或 `## 4.` 之前）
3. 在 §3 之后、§5 之前插入 §4
4. 不修改 §3 内容

### 8.5 与 ST-RA-03 的接口

ST-RA-03 产出的 Approved Improvement Input 是跟踪的起点：
- `proposal_id` → Action Item 的 `proposal_ref`
- `target_agent` → 不直接使用，跟踪侧不关心消费者
- `due_date`（来自 CA/PA Proposal）→ Action Item 的 `due_date`
- `validation_method`（来自 CA/PA Proposal）→ Effectiveness Check 的 `method` 初始值

---

## 9. 安全与性能设计

### 9.1 安全设计

| 安全约束 | 实现方式 | 验证方式 |
|---|---|---|
| 不可自动关闭 | Closure Decision engine 拒绝非 reviewer 触发的关闭 | fixture：Skill 尝试自动关闭 → 拒绝 |
| 关闭条件不可绕过 | 四个条件硬断言，decision="closed" 要求全部 true | fixture：逐一缺失条件 → 拒绝关闭 |
| 不可自动完成行动项 | Action Item 状态迁移仅由 Owner/Reviewer 触发（overdue 仅标记） | fixture：模拟自动完成 → 拒绝 |
| 不可自动判断有效性 | Effectiveness Check 的 result 仅由 reviewer 填写 | fixture：Skill 尝试填充 result → 拒绝 |
| 不修改上游数据 | SKILL.md 追加不修改 §3 已有内容 | CP6 diff 检查 |

### 9.2 权限模型

| 角色 | 权限 |
|---|---|
| 质量负责人（Owner） | 标记行动项状态、填写实施证据 |
| 人工 Reviewer | 有效性检查结果判定、关闭决策、设置观察窗豁免 |
| 测试架构师（ptm-tse） | 调用 Skill 执行 status-overview、发起有效性检查 |
| AI/Skill | 过期检测、条件汇总、复发查询；禁止：关闭/批准/完成/有效性判定 |

### 9.3 性能考虑

操作均为文件化静态查询，无性能瓶颈。复发统计涉及 SQLite 只读查询，量级为单表扫描，在常规 RA 数据量下不会有性能问题。

---

## 10. 测试设计

### 10.1 测试策略

| 测试类型 | 覆盖范围 | 验证方式 |
|---|---|---|
| 状态机测试 | Action Item 状态迁移全路径 | approach fixture |
| 门控测试 | 四关闭条件逐一缺失 | negative fixture |
| 边界测试 | overdue 临界时点、观察窗边界 | boundary fixture |
| 安全测试 | 自动关闭、自动完成、自动有效性判定 | forbidden fixture |
| Schema 测试 | 三个模板文件的必填字段 | 静态 schema 校验 |

### 10.2 测试场景清单

| ID | 场景 | 类型 | 输入 | 预期输出 |
|---|---|---|---|---|
| T-RA-04-FSM-01 | not-started → in-progress | approach | 行动项 + 开始标记 | status = "in-progress" |
| T-RA-04-FSM-02 | in-progress → done | approach | 行动项 + 完成标记 | status = "done" |
| T-RA-04-FSM-03 | not-started 过期 | boundary | 行动项 status="not-started", now > due_date | 标记 overdue |
| T-RA-04-FSM-04 | in-progress 过期 | boundary | 行动项 status="in-progress", now > due_date | 标记 overdue |
| T-RA-04-FSM-05 | done 不过期 | boundary | 行动项 status="done", now > due_date | 不过期 |
| T-RA-04-CLOSE-01 | 四条件全部满足 → 关闭 | approach | all_actions_complete=true, effectiveness_passed=true, observation_window_satisfied=true, no_same_category_recurrence=true | decision="closed" |
| T-RA-04-CLOSE-02 | 条件 1 不满足（有未完成行动项） | negative | all_actions_complete=false | decision="open" + residual_risks |
| T-RA-04-CLOSE-03 | 条件 2 不满足（有效性 failed） | negative | effectiveness_passed=false | decision="open" + residual_risks |
| T-RA-04-CLOSE-04 | 条件 3 不满足（观察窗不足） | negative | observation_window_satisfied=false | decision="open" + 提示等待 |
| T-RA-04-CLOSE-05 | 条件 4 不满足（有同类复发） | negative | recurrence_rate > 0 | decision="open" + 复发详情 |
| T-RA-04-CLOSE-06 | 全部 done 但有效性 failed → 保持开放 | boundary | all_actions_complete=true, effectiveness_passed=false | decision="open" |
| T-RA-04-SEC-01 | Skill 自动关闭被拒绝 | forbidden | Skill 尝试直接设置 decision="closed" | 拒绝，提示需 reviewer 确认 |
| T-RA-04-SEC-02 | Skill 自动完成行动项被拒绝 | forbidden | Skill 尝试直接设置 status="done" | 拒绝 |
| T-RA-04-SEC-03 | Skill 自动判定有效性被拒绝 | forbidden | Skill 尝试直接设置 result="passed" | 拒绝 |
| T-RA-04-OVERDUE-01 | 批量 overdue 检测 | approach | 3 个行动项，2 个过期 | overdue 标记正确 |
| T-RA-04-EFF-01 | 有效性检查创建 | approach | action_id + window_days=30 | check_id 生成，result="planned" |
| T-RA-04-EFF-02 | reviewer 填写有效性结果 | approach | check_id + reviewer + result="passed" | result 更新为 "passed" |
| T-RA-04-RECUR-01 | 复发率计算 | approach | same_category_count=2, total=10 | recurrence_rate=0.2 |

### 10.3 与 Feature TEST-PLAN 的对齐

| Feature TEST-PLAN ID | 本 Story 测试覆盖 | 映射 |
|---|---|---|
| T-TRK-01 | T-RA-04-FSM-01/02 | 行动项状态流转 |
| T-TRK-02 | T-RA-04-FSM-03/04/05 | 过期行动项标记 |
| T-TRK-03 | T-RA-04-CLOSE-01 | 四条件满足 → 关闭 |
| T-TRK-04 | T-RA-04-CLOSE-02/03/04/05 | 条件不足 → 保持开放 |
| T-TRK-05 | T-RA-04-EFF-01/02 | 有效性检查结果 |
| T-TRK-09 | T-RA-04-SEC-01 | 不自动关闭 RA |

---

## 11. 实施步骤

| 步骤 | 任务 | 输出 | 依赖 |
|---|---|---|---|
| 1 | 读取已有 `skills/improvement-tracker/SKILL.md`（ST-RA-03 产出） | SKILL.md 现有内容 | ST-RA-03 完成 |
| 2 | 创建 `templates/action-item.yaml`（schema + 示例 + 状态约束） | action-item.yaml | 无 |
| 3 | 创建 `templates/effectiveness-check.yaml`（schema + 示例 + 复发统计说明） | effectiveness-check.yaml | 无 |
| 4 | 创建 `templates/closure-decision.yaml`（schema + 四条件 + 示例） | closure-decision.yaml | 无 |
| 5 | 在 SKILL.md 末尾追加 §4 闭环跟踪（行动项 + 有效性 + 关闭决策） | SKILL.md §4 | 步骤 1 |
| 6 | 在 SKILL.md §4.4 写入关闭条件不可绕过规则 | SKILL.md §4.4 | 步骤 5 |
| 7 | 创建 Fixture 数据（action items、effectiveness checks、closure decisions 各种场景） | Fixture 数据 | 步骤 2-6 |
| 8 | 本地静态验证：schema 校验、状态机路径、关闭条件全组合 | 验证记录 | 全部 |

---

## 12. 风险、难点与预研建议

### 12.1 风险

| 风险 ID | 描述 | 概率 | 影响 | 缓解措施 |
|---|---|---|---|---|
| R-LLD-RA-04-01 | ST-RA-03 未完成时无法追加 SKILL.md，阻塞串行链 | 低 | 高 | CP5 全量确认后按严格串行执行；本 LLD 定义好追加位置锚点和接口 |
| R-LLD-RA-04-02 | 复发统计依赖 SQLite measure_link 表，若 SCHEMA 未包含 proposal_ref 字段则查询失败 | 中 | 中 | 在 ST-RA-INGEST-DB LLD 中确认 measure_link schema 包含 proposal_ref；若缺失则通过 OPEN 项追回 |
| R-LLD-RA-04-03 | 观察窗 30 天的默认值可能因业务类型不同需调整 | 低 | 低 | 设计支持 reviewer 手动调整 window_days，默认值为建议值 |

### 12.2 难点

1. **串行追加 SKILL.md 的正确性保证**：三个 Story 写入同一文件的风险。方案：使用显式分节标记（§3/§4/§5）+ 追加位置锚点，CP6 时通过 diff 验证每次追加未改动已有节。

2. **关闭条件"无同类复发"的定义歧义**：`same_category` 的粒度（产品级、模块级、严重度级）可能影响复发判断。方案：首版以 `proposal_ref` 为粒度，同 proposal 下的问题单计为同类；若需更细粒度，由 reviewer 在复查时调整。

### 12.3 预研建议

- 确认 SQLite `measure_link` 表 schema 是否包含 `proposal_ref` 字段；如不包含，需由 ST-RA-INGEST-DB 追加。

### 开放项

| ID | 主题 | 状态 | 影响 | 重访条件 |
|---|---|---|---|---|
| OPEN-RA-04-01 | `measure_link` 表的 `proposal_ref` 字段确认 | OPEN | 影响复发统计查询 | ST-RA-INGEST-DB LLD 确认后关闭 |
| OPEN-RA-04-02 | 观察窗豁免机制（reviewer 手动设置 window_satisfied=true）的权限边界 | OPEN | 可能被滥用绕过观察窗要求 | CP7 fixture 中验证豁免需 reviewer 身份 |

---

## 13. 回滚与发布策略

### 13.1 回滚策略

| 回滚场景 | 回滚操作 | 影响范围 |
|---|---|---|
| SKILL.md §4 逻辑错误 | git revert 或手动移除 §4 内容，不影响 §3 | improvement-tracker 跟踪侧降级 |
| action-item.yaml 等模板 schema 错误 | 修正模板文件 | 仅影响新创建的 action items |
| 关闭决策历史记录有误 | 修正 closure-decision.yaml 模板，已生成文件保留 | 已生成文件不变 |
| 需要完全下线跟踪侧 | 删除 SKILL.md §4 内容，移除 3 个模板文件 | improvement-tracker Skill 仅保留 CA/PA 侧 |

### 13.2 发布策略

本 Story 随 CR-030 Wave 3 一起发布，与 ST-RA-03 和 ST-RA-06.3-TRACK 共同构成完整的 improvement-tracker Skill。不能独立交付。ST-RA-03 → ST-RA-04 的串行依赖决定了发布顺序。

---

## 14. Definition of Done

- [ ] `skills/improvement-tracker/SKILL.md` §4 闭环跟踪完整
  - [ ] §4.1 Action Item 状态机定义
  - [ ] §4.2 Effectiveness Check 流程
  - [ ] §4.3 Closure Decision 四条件
  - [ ] §4.4 关闭条件不可绕过规则
- [ ] `skills/improvement-tracker/templates/action-item.yaml` 存在，含完整 schema + 状态枚举
- [ ] `skills/improvement-tracker/templates/effectiveness-check.yaml` 存在，含复发统计字段
- [ ] `skills/improvement-tracker/templates/closure-decision.yaml` 存在，含四条件 + 示例
- [ ] Action Item 状态机正确流转（not-started→in-progress→done，overdue 自动标记）
- [ ] 四关闭条件全部满足时 decision="closed"
- [ ] 任一条件不满足时 decision="open" + residual_risks + follow_up_actions
- [ ] 行动项全部 done 但有效性 failed → 保持开放（fixture 证明）
- [ ] 不可自动关闭（fixture 拒绝 Skill 的自动关闭）
- [ ] 不可自动完成行动项或判定有效性
- [ ] SKILL.md 追加操作未修改 ST-RA-03 写入的 §3 内容
- [ ] DEV-LOG.md 追加记录

## 15. 实现灰区与取舍记录

| 灰区 ID | 问题 | 决策 | 理由 | 影响面 |
|---|---|---|---|---|
| GY-RA-04-01 | overdue 是否持久化还是动态计算 | 动态计算，不修改持久化文件 | 避免频繁写入；过期状态是时间函数 | 每次查询需重新计算，但量级不影响 |
| GY-RA-04-02 | 观察窗豁免（reviewer 手动设置 window_satisfied=true）的权限 | 允许 reviewer 豁免，需记录豁免原因 | 业务场景中可能存在观察期已过但未记录起始时间的情况 | 豁免记录写入 effectiveness-check notes |
| GY-RA-04-03 | 复发统计的同类定义 | 首版以 proposal_ref 为粒度 | 简单明确，与 CA/PA 一一对应 | 后续可扩展为更细粒度（如按 root_cause 分类） |

---

## 16. 与相邻模块的集成契约

### 16.1 上游（ST-RA-03 改进输入治理）

ST-RA-04 依赖 ST-RA-03 产出。CA/PA Proposal 和 Approved Improvement Input 是跟踪的起点：
- `proposal_id` 贯穿 Action Item → Effectiveness Check → Closure Decision 全链路
- `validation_method`（CA/PA Proposal）→ Effectiveness Check 的 `method` 初始值
- `due_date`（CA/PA Proposal）→ Action Item 的 `due_date`

### 16.2 下游（ST-RA-06.3-TRACK 措施基线）

ST-RA-04 的 Closure Decision 产出（特别是 `decision = "open"` 时的 `residual_risks` 和 `follow_up_actions`）可被 ST-RA-06.3-TRACK 的措施刷新逻辑读取，用于触发 MeasureBaseline 刷新提示。

### 16.3 上游（SQLite measure_link）

ST-RA-04 的 Effectiveness Check 中复发统计依赖 SQLite `measure_link` 表查询。这是只读依赖，不影响 SQLite 数据。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-16 | host-orchestrator | CP5 B12：迁移为当前 full-lld 证据兼容格式，增加 canonical Story ID 与 §0 工程依据/章节索引；不改变原设计契约。 |
