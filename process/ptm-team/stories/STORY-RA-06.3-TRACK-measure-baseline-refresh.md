---
story_id: "ST-RA-06.3-TRACK"
title: "措施基线管理与刷新提示"
status: "ready-for-verification"
priority: "P1"
wave: 4
source_cr: "CR-030"
source_story: "ST-RA-06"
feature: "FEAT-RA-TRACKING"
feature_design_refs:
  - "docs/features/feat-ra-tracking/DESIGN.md"
lld_policy:
  required_level: "technical-note"
  trigger_reasons: []
  rationale: "MeasureBaseline 管理 + 刷新提示；逻辑相对独立"
depends_on:
  - "ST-RA-04"
  - "ST-RA-06.1-DETECT"
output_files:
  - "skills/improvement-tracker/SKILL.md"
  - "skills/improvement-tracker/templates/measure-baseline.yaml"
created_by: "meta-se"
created_at: "2026-07-16"
---

# ST-RA-06.3-TRACK: 措施基线管理与刷新提示

## dev_context

## 技术说明

> **lld_policy**: technical-note
> **设计证据类型**: Story 卡片内嵌技术说明
> **所属 Feature**: FEAT-RA-TRACKING
> **依赖**: ST-RA-04（闭环跟踪模型）, ST-RA-06.1-DETECT（S2 变更检测）
> **写入文件**: `skills/improvement-tracker/SKILL.md` §5, `skills/improvement-tracker/templates/measure-baseline.yaml`
> **CP5 评审意见 H3**：MeasureBaseline 是 HLD 七项可信治理契约之一，本技术说明需显式引用正向 LLD 的精确覆盖，证明"逻辑相对独立"豁免 full-lld 有据可查。
>
> **正向 LLD 覆盖追溯**：
>
> | 覆盖项 | 正向 LLD 位置 | 章节 | 本 Story 的增量 |
> |-------|-------------|------|---------------|
> | MeasureBaseline schema 定义 | `ST-RA-03-LLD.md` | §3.2 CA/PA Proposal（关联 proposal_ref）| 新增 baseline 版本、观察窗、刷新提示字段 |
> | 措施审批门控 | `ST-RA-03-LLD.md` | §5 状态机（draft→review→approved）| 新增 needs-baseline→active 转换（需 reviewer 确认） |
> | 行动项与有效性 | `ST-RA-04-LLD.md` | §5 行动项状态机、§4 关闭四条件 | 增加事件驱动的 proposed_status 提示 |
> | S2 变更检测 | `ST-RA-06.1-DETECT-LLD.md` | §4 change_set、§12 集成契约 | 消费 change_set 判断同性复发 |
> | S2 增量重算 | `ST-RA-06.2-REFRESH-LLD.md` | §4 受影响维度、§12 差异报告格式 | 消费差异报告判断措施受影响范围 |
> | 敏感字段策略 | `ST-RA-05.2-CLEAN-LLD.md` | §2.6 QCOMB 规则 | MeasureBaseline 字段全为 cleaned/report 级，无 raw 字段 |

### 1. 设计依据

本 Story 实现 HLD REV-03 中 `MeasureBaseline` 契约（7 项可信治理契约之一）：

> *"`MeasureBaseline`：刷新关联措施版本、范围、审批/实施/有效性证据和观察窗。无基线 = `needs-baseline`，不判失效。"*

以及 REQ-RA-022（让新数据触发对既有 CA/PA 候选和已批准措施的状态刷新与重新评估提示）。设计来源：

- `docs/design/HLD.md` REV-03 §可信分析治理约束 — MeasureBaseline 契约
- `docs/features/feat-ra-tracking/DESIGN.md` §2.4 MeasureBaseline, §4 S2 措施刷新规则
- `docs/features/feat-ra-tracking/TASKS.md` ST-RA-06.3-TRACK 任务清单

### 2. 文件影响范围

| 文件 | 操作 | 变更描述 |
|---|---|---|
| `skills/improvement-tracker/SKILL.md` | 追加 §5 | 在 ST-RA-03（§3 CA/PA 侧）和 ST-RA-04（§4 跟踪侧）之后，追加 §5 MeasureBaseline 管理与 S2 措施刷新逻辑 |
| `skills/improvement-tracker/templates/measure-baseline.yaml` | 创建 | MeasureBaseline schema + 刷新状态枚举 + 示例 |

**写入顺序**：`improvement-tracker/SKILL.md` 串行写入：ST-RA-03 → ST-RA-04 → ST-RA-06.3-TRACK。本 Story 不修改 §3 和 §4 的内容。

### 3. 接口、数据与权限变化

#### 3.1 MeasureBaseline Schema

```yaml
baseline_id: string          # 唯一标识，格式：BASELINE-{proposal_id}-{version}
proposal_ref: string         # 关联 CA/PA Proposal ID
version: int                 # 基线版本号（首次为 1，每次刷新递增）
scope: string                # 措施适用范围
approval_ref:
  reviewer: string           # 批准人
  approved_at: datetime
implementation_evidence: string  # 实施证据引用
effectiveness_evidence: string   # 有效性证据引用
observation_window:
  start: date
  end: date
status: enum                 # 正式状态（仅 reviewer 可变更）: 'needs-baseline' | 'active' | 'completed' | 'needs-review' | 'superseded'
proposed_status: enum        # 系统建议状态（只读提示）: 'needs-baseline' | 'active' | 'completed' | 'needs-review' | 'superseded' | null
refresh_hint: string         # 刷新原因人类可读描述（如"新问题可能影响措施有效性"）
refreshed_at: datetime       # 提示生成时间
refreshed_by: string         # 提示触发来源（事件类型）
refresh_reason: string       # 刷新原因摘要
created_at: datetime
```

#### 3.2 措施刷新规则

> ⚠️ CP5 评审意见 H2：HLD 只允许自动生成刷新提示（"保持/完成/需复核/失效"），reviewer 是唯一状态变更者。
> 系统产出 `proposed_status` / `refresh_hint`，不自动创建 baseline 或修改正式状态。

| 触发条件 | proposed_status | refresh_hint | 触发来源 | 需要 reviewer 确认的操作 |
|---|---|---|---|---|
| 关联 proposal 无 MeasureBaseline | `needs-baseline` | "需先建立措施基线" | S2 变更检测首次关联 | Reviewer 审查后创建 baseline → status 变为 `active` |
| S2 检测到新相关问题单（`same_category_count` 增加） | `needs-review`（不自动改 status） | "新问题可能影响措施有效性" | ST-RA-06.2-REFRESH 差异报告 | Reviewer 审查后决定：保持 active 或标记 needs-review |
| ST-RA-04 行动项全部 done | `completed`（不自动改 status） | "可进入观察期" | ST-RA-04 闭环跟踪事件 | Reviewer 确认各项 done 后标记 completed |
| ST-RA-04 观察窗通过 + 无复发 | `active`（不自动改 status） | "观察窗已满足，措施持续有效" | ST-RA-04 闭环跟踪事件 | Reviewer 确认观察窗和复发检查后保持 active |
| S2 检测到同类问题复发 | `needs-review`（不自动改 status） | "同类复发，措施可能失效" | ST-RA-06.2-REFRESH 差异报告 | Reviewer 审查后决定：保持 active 或标记 needs-review |
| 措施被新的 CA/PA 取代 | `superseded`（不自动改 status） | 被新 proposal_id 取代 | S2 增量重算 | Reviewer 确认后标记 superseded |

#### 3.3 权限模型（修正：reviewer 是唯一状态变更者）

| 操作 | 触发者 | 权限 |
|---|---|---|
| 检测到事件并生成 proposed_status + refresh_hint | 系统 | 自动（只写入提示字段，不修改正式 status） |
| 创建 MeasureBaseline | **人工 Reviewer** | 需手动确认（基于系统 proposed_status=needs-baseline 提示） |
| 正式 status 变更（→ needs-review/completed/superseded） | **人工 Reviewer** | 需手动确认（基于系统 refresh_hint） |
| status 改回 active | **人工 Reviewer** | 需手动确认 |
| 改版 scope/approval_ref | **人工 Reviewer** | 需重新批准 |
| **禁止**：系统自动改变正式 status | 所有 | 不允许（只能写 proposed_status/hint） |
| **禁止**：系统自动创建 MeasureBaseline | 所有 | 不允许（只能提示 needs-baseline） |
| **禁止**：自动修改下游任务 | 所有 | 不允许 |
| **禁止**：无基线时判措施失效 | 所有 | 不允许（只标 needs-baseline） |

#### 3.4 上游读取依赖

| 上游对象 | 提供者 | 用途 |
|---|---|---|
| CA/PA Proposal（approved） | ST-RA-03 | 通过 proposal_ref 关联 |
| Action Item 状态 | ST-RA-04 | 判断是否"措施实施完成" |
| Closure Decision | ST-RA-04 | 判断是否"观察窗通过 + 无复发" |
| S2 差异报告 | ST-RA-06.2-REFRESH | 检测新增问题单和同类复发 |

### 4. 核心流程

```
[事件触发 → 系统产出提示]

      ├── S2 变更检测 → 新问题单
      │     │
      │     ├── 关联 proposal 有 baseline？
      │     │     ├── YES → proposed_status = "needs-review"
      │     │     │         refresh_hint = "新问题可能影响措施有效性"
      │     │     └── NO  → proposed_status = "needs-baseline"
      │     │               refresh_hint = "需先建立措施基线"
      │
      ├── ST-RA-04 行动项全部 done
      │     │
      │     └── proposed_status = "completed"
      │           refresh_hint = "可进入观察期"
      │
      ├── ST-RA-04 关闭条件满足（观察窗通过 + 无复发）
      │     │
      │     └── proposed_status = "active"
      │           refresh_hint = "观察窗已满足，措施持续有效"
      │
      └── S2 检测到同类复发
            │
            └── proposed_status = "needs-review"
                  refresh_hint = "同类复发，措施可能失效"

[人工 Reviewer 确认操作]  ← 唯一状态变更者
      │
      ├── 审查 needs-baseline → 创建 MeasureBaseline → status = "active"
      ├── 审查 needs-review → 保持 active 或确认 needs-review
      ├── 审查 completed → 确认并标记 completed
      ├── 审查 superseded → 确认并标记 superseded
      └── 创建新版本 baseline（更新 scope/approval_ref）
```

### 5. 错误处理与降级

| 场景 | 处理方式 |
|---|---|
| proposal_ref 指向不存在的 CA/PA | 记录异常日志，不创建 baseline |
| 同时多个事件触发刷新 | 最后一次 wins，但保留所有 refresh_reason 记录 |
| 上游数据不可用（ST-RA-04/06.2 未产出） | 标记 needs-baseline，等待上游完成后回填 |
| SKILL.md §5 追加位置冲突 | 如果 §4 或 §5 标记已被后续 Story 修改，报错并停止 |

### 6. 测试入口

| 测试 ID | 场景 | 预期 | 来源 Feature TEST-PLAN |
|---|---|---|---|
| T-TRK-06 | MeasureBaseline 创建 | baseline 含 version/scope/approval_ref/observation_window | T-TRK-06 |
| T-TRK-07 | 无基线 → needs-baseline | 不判措施失效，输出提示 | T-TRK-07 |
| T-TRK-08 | 措施刷新提示 | needs-review/completed/needs-baseline 只产出 proposed_status+refresh_hint，不自动变更正式 status | T-TRK-08 |
| T-TRK-10 | 不自动改变批准状态 | 措施刷新不改变 approval_status | T-TRK-10 |
| T-RA-063-BASELINE-01 | S2 新问题单 → needs-review 提示 | proposal 有新问题单，proposed_status=needs-review，正式 status 不变 | 本 Story 新增 |
| T-RA-063-BASELINE-02 | 行动项完成 → completed 提示 | 所有 action items done，proposed_status=completed，正式 status 不变 | 本 Story 新增 |
| T-RA-063-BASELINE-03 | 观察窗通过 → active 提示 | 关闭条件满足，proposed_status=active，正式 status 不变 | 本 Story 新增 |
| T-RA-063-BASELINE-04 | 同类复发 → needs-review 提示 | 复发检测，proposed_status=needs-review，正式 status 不变 | 本 Story 新增 |
| T-RA-063-GATE-01 | 不自动改变 AS | fixture 模拟刷新触发 approval_status 变更 → 拒绝 | 安全 |
| T-RA-063-GATE-02 | 不自动修改下游任务 | fixture 模拟刷新触发下游写入 → 拒绝 | 安全 |

### 7. 风险与重访条件

| 风险 | 缓解措施 | 重访条件 |
|---|---|---|
| MeasureBaseline 刷新提示可能产生噪音（频繁 needs-review） | 只标记不行动，reviewer 手动确认 | 3 个以上 reviewer 反馈噪音过多 |
| `needs-review` 和 `active` 之间的来回可能失去意义 | 记录 refresh_reason 历史，reviewer 可查看变化轨迹 | CP7 时审查刷新频率和 reviewer 反馈 |
| 与 ST-RA-04 和 ST-RA-06.2-REFRESH 的事件传递路径不明确 | 本 LLD 明确读取上游产出（文件化事件），不依赖运行时事件 | CP6 实现时确认文件路径约定 |

### 8. 偏离记录

| 偏离项 | 偏离原因 | 影响 |
|---|---|---|
| HLD 中提到"措施刷新只产生'保持/完成/需复核/失效'提示"，本 Story 使用 `needs-baseline/active/completed/needs-review/superseded` 五种状态 | 需要与 FEAT-RA-TRACKING DESIGN 中的具体状态枚举一致；"保持"→`active`，"需复核"→`needs-review`，"失效"→不主动判断 | 无功能影响，仅命名对齐 |
| 首版不实现"关联措施版本"的自动版本号递增 | 需要多轮 S2 增量重算后才需要版本管理；首版 version 固定为 1 | 后续 Story S2 迭代时补充 |
| **CP5 评审 H2 整改**：v1.0 允许系统自动创建 baseline + 自动变更 status → v1.1 改为 proposed_status/refresh_hint，所有正式 status 和 baseline 创建由 reviewer 确认 | HLD 只允许自动生成提示，reviewer 是唯一状态变更者 | 增加人工确认步骤，消除自动状态变更越权 |


## validation_context
- 关键验证：无基线 → needs-baseline；新增问题单 → needs-review；不自动改变批准状态

## acceptance_criteria
1. MeasureBaseline 含版本/范围/审批引用/观察窗
2. 无基线 → needs-baseline（不判措施失效）
3. 措施刷新只产出提示，不自动改变批准状态
4. 不自动修改下游任务
5. measure-baseline 模板完整
