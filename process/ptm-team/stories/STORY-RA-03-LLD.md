---
story_id: "STORY-RA-03"
canonical_story_id: "ST-RA-03"
title: "改进输入治理（improvement-tracker CA/PA 侧）"
lld_policy: "full-lld"
tier: "M"
feature: "FEAT-RA-IMPROVEMENT"
design_evidence_type: "full-lld"
source_hld: "docs/design/HLD.md (v1.2, REV-03)"
source_feature_design: "docs/features/feat-ra-improvement/DESIGN.md (v1.0)"
depends_on: ["ST-RA-05.3-ANALYZE"]
shared_with: ["ST-RA-04", "ST-RA-06.3-TRACK"]
created_by: "meta-dev"
created_at: "2026-07-16"
---

# ST-RA-03 高层设计（LLD）：改进输入治理

## 0. 工程依据与模板索引

| 来源 | 消费内容 |
|---|---|
| HLD / Feature DESIGN / Feature Matrix | CA/PA 人工审批边界和 `full-lld` 约束 |
| Story `ST-RA-03` | 输出文件、验收标准和依赖 |

本文保留原有详细章节；目标、需求、模块拆分、代码结构、数据模型、API、流程、技术细节、安全、测试、实施、风险与 DoD 分别由后续编号章节定义。

> 对应 HLD REV-03 的 `improvement-tracker` Skill CA/PA 侧。
> 本章 LLD 证据覆盖 14 个语义要点。

## LLD 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-16 | meta-dev | 初始 LLD：CA/PA 草案生成、批准状态机、Approved Improvement Input、消费者映射、文件影响、测试设计 |

---

## 1. Goal

基于已确认的分析结论（`analysis-confirmed` 状态的 RA Report），生成结构化的 CA/PA 草案，经人工 reviewer 批准后产出不可变的 Approved Improvement Input，供下游 Agent 只读消费。确保未批准不分发、批准后可追溯，且不自动操作下游。

**核心成功标准：**

| 指标 | 目标值 | 验证方式 |
|---|---|---|
| 未批准不分发 | 未确认 RA / 未批准 CA/PA 的 Approved Input 数为 0 | fixture 边界断言 |
| CA/PA 完整性 | 每项含依据、类型、目标、Owner、验收、副作用 | schema 校验 + 必填字段 fixture |
| Approved Input 可追溯 | 100% 含 source_ra、proposal_id、approval_ref | schema 校验 |

---

## 2. Requirements

### 2.1 Functional Requirements

| 需求 ID | 描述 | 本 Story 覆盖 |
|---|---|---|
| REQ-RA-007 | 让改进 Owner 获得可执行的 CA/PA 草案及其 Owner、期限、验证和副作用信息 | 完全覆盖：CA/PA Proposal 草案生成 |
| REQ-RA-008 | 让下游负责人只收到经过批准、可行动且可追溯的改进输入 | 完全覆盖：Approved Improvement Input 生成门控 |
| REQ-RA-018 | 输出可审查的纠正/预防改进措施候选，未批准不得分发或自动执行 | 完全覆盖：批准状态机 |

### 2.2 Non-Functional Requirements

| 质量属性 | 设计目标 | 实现手段 |
|---|---|---|
| 可审计性 | 100% 批准追溯 | proposal_id → approval_ref → approved-input 链路 |
| 安全性 | deny-by-default | 未批准门控，不自动写入下游 |
| 不可变性 | Approved Input 生成后不可修改 | 生成即冻结，修改需 reviewer 创建新版本 |
| 互操作性 | 跨 Agent 只读消费 | 标准化 YAML schema，target_agent 字段映射 |

### 2.3 Scope Boundary

| 负责事项 | 不负责事项 |
|---|---|
| CA/PA 草案基于已确认分析生成 | 分析结论本身（由 F-021 负责） |
| 批准状态机（draft → approved/rejected） | 下游任务创建 |
| Approved Input 的 immutable 生成 | 下游 Agent 文件的直接修改 |
| 消费者映射（target_agent → consumer_status） | 下游消费结果跟踪 |

---

## 3. 模块拆分与职责

本 Story 创建 `improvement-tracker` Skill 的 CA/PA 侧，按职责分为 4 个子模块：

| 模块 | 职责 | 输入 | 输出 | 禁止行为 |
|---|---|---|---|---|
| **CA/PA 草案生成器** | 基于已确认 RA Report 生成 CA/PA Proposal 草案 | 已确认 RA Report（`analysis-confirmed`） | `capa-proposal.yaml` | 未确认 RA → 拒绝生成 |
| **批准状态机** | CA/PA 草案的批准/拒绝流转 | CA/PA 草案 + reviewer 决定 | 更新 approval_status 和 approval_ref | 自动批准、自动分发 |
| **Approved Input 生成器** | 批准后生成不可变的 Approved Improvement Input | 已批准 CA/PA Proposal | `approved-input.yaml` | 未批准生成、生成后修改 |
| **消费者映射器** | 将 Approved Input 映射到目标消费者 Agent | target_agent 字段 | consumer_status 标记 | 自动创建下游任务 |

**Skill 目录结构（本次创建）：**

```
skills/improvement-tracker/
├── SKILL.md                    # Skill 定义（CA/PA 侧）
└── templates/
    ├── capa-proposal.yaml      # CA/PA 草案模板
    └── approved-input.yaml     # Approved Improvement Input 模板
```

---

## 4. 代码结构与文件影响范围

### 4.1 文件清单

| 文件 | 操作 | 写入模块 | 说明 |
|---|---|---|---|
| `skills/improvement-tracker/SKILL.md` | 创建（写 CA/PA 侧） | 所有子模块 | Skill 顶层定义、触发词、流程、状态机、门控规则 |
| `skills/improvement-tracker/templates/capa-proposal.yaml` | 创建 | CA/PA 草案生成器 | 草案 schema 与示例 |
| `skills/improvement-tracker/templates/approved-input.yaml` | 创建 | Approved Input 生成器 | 不可变输入 schema 与示例 |

### 4.2 文件所有权

| 文件 | owner | shared_with | write_section |
|---|---|---|---|
| `skills/improvement-tracker/SKILL.md` | FEAT-RA-IMPROVEMENT | ST-RA-04, ST-RA-06.3-TRACK | CA/PA creation, approval gate, approved input |
| `skills/improvement-tracker/templates/capa-proposal.yaml` | FEAT-RA-IMPROVEMENT | — | 独占 |
| `skills/improvement-tracker/templates/approved-input.yaml` | FEAT-RA-IMPROVEMENT | — | 独占 |

**串行约束**：`improvement-tracker/SKILL.md` 由 ST-RA-03 → ST-RA-04 → ST-RA-06.3-TRACK 顺序写入。本文档写入 CA/PA 侧定义，ST-RA-04 追加跟踪侧，ST-RA-06.3-TRACK 追加 MeasureBaseline 侧。

### 4.3 不涉及的文件

以下文件不属于本 Story 范围：
- `skills/itr-ticket-ingestion/*`（F-020 专有）
- `skills/reverse-analysis/*`（F-021 专有）
- `data/*`（SQLite，只读消费 ticket 数据由上游 Story 负责）
- `agents/ptm-tse.md`（编排更新由 ST-RA-05.3-ANALYZE 负责）

---

## 5. 数据模型与持久化设计

### 5.1 CA/PA Proposal Schema

```yaml
# capa-proposal.yaml — 本 Story 创建
proposal_id: string        # 唯一标识，格式：PROPOSAL-{timestamp}-{seq}
analysis_ref:
  ra_report_id: string     # 关联 RA Report ID
  analysis_run_id: string  # 关联 analysis_run ID
kind: enum                 # 'corrective' | 'preventive'
title: string              # CA/PA 标题
basis: string              # 依据（引用分析结论）
target: string             # 改进目标
owner: string              # 单一 Owner
due_date: date             # 到期日期
priority: enum             # 'P0' | 'P1' | 'P2'
validation_method: string  # 验证方式
side_effects: string       # 潜在副作用
approval_status: enum      # 'draft' | 'approved' | 'rejected'
approval_ref:
  reviewer: string         # 批准人
  approved_at: datetime    # 批准时间
  comments: string         # 审批意见
created_at: datetime
updated_at: datetime
```

**约束：**
- `proposal_id` 全局唯一，不可变
- `kind`、`title`、`basis`、`target`、`owner`、`due_date`、`priority`、`validation_method` 为必填
- `approval_status` 仅可由人工 reviewer 改变
- `approval_ref` 仅在 `approval_status=approved` 时必填

### 5.2 Approved Improvement Input Schema

```yaml
# approved-input.yaml — 本 Story 创建
input_id: string           # 唯一标识，格式：INPUT-{timestamp}-{seq}
source_ra: string          # 来源 RA Report ID
proposal_id: string        # 关联 CA/PA Proposal ID
kind: enum                 # 'corrective' | 'preventive'
title: string              # 改进标题
target_agent: enum         # 目标消费 Agent：'ptm-tde' | 'ptm-te' | 'ptm-tae' | 'ptm-qa'
scope: string              # 适用范围
acceptance_criteria: string # 验收标准
priority: enum             # 'P0' | 'P1' | 'P2'
constraints: string        # 限制条件
approval_ref:
  reviewer: string
  approved_at: datetime
consumer_status: enum      # 'pending-consumer' | 'consumed' | 'blocked'
created_at: datetime
immutable: true            # 强制不可变标记
```

**约束：**
- `input_id` 全局唯一，不可变
- 必须在关联 `proposal_id` 的 `approval_status=approved` 时才能生成
- 生成后所有字段不可变（`immutable=true`）
- `target_agent` 枚举值限定为现有 Agent 列表
- `consumer_status` 默认值为 `pending-consumer`

### 5.3 持久化说明

本 Story 的 CA/PA Proposal 和 Approved Improvement Input 以文件化工件形式持久化（YAML 模板渲染）。各对象的生命周期管理由 `improvement-tracker` Skill 的 Skill 内规则控制。不依赖 SQLite 或其他数据库—这与 HLD REV-03 的设计一致：SQLite 仅由 F-020（INGESTION）写入 ticket/change_history 等规范化数据，improvement-tracker 产物是文件化治理对象。

当未来需要跨 Feature 查询措施状态时，可考虑通过 `measure_link` 表（F-023 负责）关联 SQLite，但本 Story 不引入 SQLite 写入。

---

## 6. API / Interface 设计

### 6.1 improvement-tracker Skill（CA/PA 侧）接口

由于 `improvement-tracker` 是 Skill 而非独立服务，接口为 Skill 触发词 + 结构化输入/输出契约。

**触发条件：**
- Skill 名称：`improvement-tracker`
- 触发词：`improvement-tracker`、`改进跟踪`、`CA/PA`、`改进治理`
- 前提：已有 `analysis-confirmed` 状态的 RA Report 可用

**执行模式：**

| 模式 | 触发描述 | 输入 | 输出 |
|---|---|---|---|
| `generate-proposal` | 基于已确认分析生成 CA/PA 草案 | RA Report ID + analysis_run ID | `capa-proposal.yaml` 草案 |
| `review-proposal` | reviewer 批准/拒绝 CA/PA 草案 | proposal_id + 审批决定 | 更新 approval_status 和 approval_ref |
| `publish-input` | 批准后生成 Approved Input | proposal_id（已批准） | `approved-input.yaml` |
| `check-gate` | 门控检查（未确认 RA / 未批准 CA/PA） | analysis_ref / proposal_id | 拒绝原因或通过 |

### 6.2 上游依赖接口

| 上游对象 | 提供者 | 本 Story 消费方式 | 必须字段 |
|---|---|---|---|
| RA Report（analysis-confirmed） | `reverse-analysis` Skill | 只读引用 | ra_report_id, analysis_status=analysis-confirmed, confirmed_by, confirmed_at |
| analysis_run | `reverse-analysis` Skill | 只读引用 | analysis_run_id, run_version |

### 6.3 下游消费接口

| 下游 Agent | 消费方式 | 消费对象 | 字段消费 |
|---|---|---|---|
| ptm-tde | 只读文件读取 | `approved-input.yaml` | scope, acceptance_criteria, constraints, priority |
| ptm-te | 只读文件读取 | `approved-input.yaml` | scope, acceptance_criteria, constraints, priority |
| ptm-tae | 只读文件读取 | `approved-input.yaml` | scope, acceptance_criteria, constraints, priority |
| ptm-qa | 只读文件读取 | `approved-input.yaml` | scope, acceptance_criteria, constraints, priority |

**下游消费约束：**
- 下游只读，不得修改 Approved Input
- 下游只能消费 `consumer_status != blocked` 的 Input
- 下游处理完成后可回写 `consumer_status: consumed`（通过 improvement-tracker 更新），但不对 Input 内容做任何修改

### 6.4 输入契约

从 `reverse-analysis` 接收的 RA Report 必须满足：

```yaml
# 必须条件
analysis_status: "analysis-confirmed"  # 人工 reviewer 已确认分析结论
confirmed_by: string                   # 非空
confirmed_at: datetime                 # 非空
# 必须数据
ra_report_id: string                   # 非空
root_cause_analysis: object            # 含 confirmed root cause
escape_analysis: object                # 含 confirmed escape points
```

不满足时必须返回拒绝原因，不生成 CA/PA 草案。

---

## 7. 核心处理流程

### 7.1 主流程

```
已确认 RA Report
      │
      ▼
[1] 前置校验：analysis_status != "analysis-confirmed"？
      │ YES → 拒绝生成，提示"分析结论未确认"
      │ NO  → 继续
      ▼
[2] 生成 CA/PA 草案
      │ 提取分析结论中的改进维度
      │ 填充 proposal schema（basis, target, owner, validation_method 等）
      │ approval_status = "draft"
      │
      ▼
[3] CA/PA 草案呈现给 reviewer
      │
      ├── reviewer 批准
      │     approval_status = "approved"
      │     approval_ref.{reviewer, approved_at, comments} 填充
      │     │
      │     ▼
      │   [4] 生成 Approved Improvement Input
      │     从已批准 proposal 提取：kind, title, scope, acceptance_criteria, priority
      │     映射 target_agent → consumer_status = "pending-consumer"
      │     │
      │     ▼
      │   [5] 输出 approved-input.yaml（不可变）
      │
      └── reviewer 拒绝
            approval_status = "rejected"
            approval_ref.comments 记录拒绝原因
            草案保留，不删除，不生成 Approved Input
```

### 7.2 消费者映射流程

```
[5] 输出 approved-input.yaml 后
      │
      ▼
target_agent 字段解析
      │
      ├── target_agent = "ptm-tde"  → 消费者存在，consumer_status = "pending-consumer"
      ├── target_agent = "ptm-te"   → 消费者存在，consumer_status = "pending-consumer"
      ├── target_agent = "ptm-tae"  → 消费者存在，consumer_status = "pending-consumer"
      ├── target_agent = "ptm-qa"   → 消费者存在，consumer_status = "pending-consumer"
      └── 未识别的 target_agent     → consumer_status = "blocked"，原因说明
```

**注**：本 Story 首版不检查下游是否真正就绪（无运行时能力）；`blocked` 仅因 target_agent 未识别触发。下游就绪检测是未来的运行时增强（需独立 runtime CR）。

### 7.3 异常路径

| 场景 | 触发条件 | 行为 | 输出 |
|---|---|---|---|
| 未确认 RA | `analysis_status != "analysis-confirmed"` | 拒绝生成 CA/PA 草案 | 拒绝原因："分析结论未确认，请先由 reviewer 确认 RA Report" |
| 缺少必填字段 | proposal 草案中 basis/target/owner 等为空 | 拒绝生成草案 | 提示缺失字段列表 |
| 未批准生成 Input | `approval_status != "approved"` | 拒绝生成 Approved Input | 拒绝原因："CA/PA 未批准，无法生成改进输入" |
| 重复 proposal_id | 同一 analysis_ref 已有关联 draft/approved proposal | 提示已有草案 | 返回已有 proposal 引用 |
| target_agent 无效 | target_agent 不在枚举值中 | consumer_status = "blocked" | blocked 原因："未识别的消费者 Agent" |

---

## 8. 技术设计细节

### 8.1 批准状态机

```
                 ┌─── reviewer approve ───► approved ───► [可生成 Approved Input]
                 │
draft ──────────┤
                 │
                 └─── reviewer reject ────► rejected ───► [保留，不生成 Input，不出现在 active 候选]
```

**状态迁移规则：**

| 源状态 | 目标状态 | 触发者 | 条件 |
|---|---|---|---|
| `draft` | `approved` | 人工 reviewer | 提供 reviewer 身份 + 审批意见（comments 可选） |
| `draft` | `rejected` | 人工 reviewer | 提供 reviewer 身份 + 拒绝理由（comments 必填） |
| `approved` | — | — | 不可回退为 draft/rejected |
| `rejected` | — | — | 不可回退为 draft/approved |

**强制约束：**
- 只有人工 reviewer 能改变 `approval_status`
- 系统（Skill/AI）不得自动批准或拒绝
- `approved` 和 `rejected` 是终态，不可回退
- 若需修改已批准的 CA/PA，必须创建新 proposal（新 proposal_id）并重新走批准流程

### 8.2 Approved Input 不可变性

Approved Improvement Input 生成后所有字段不可变。实现方式：

1. 生成时写入 `immutable: true`
2. 生成后通过 Skill 规则拒绝任何修改操作
3. 若需更新，人工 reviewer 通过创建新 proposal → 批准 → 生成新 Approved Input 实现
4. 旧 Approved Input 保留，`consumer_status` 更新为 `superseded`（由 ST-RA-06.3-TRACK 负责）

### 8.3 消费者映射逻辑

```text
target_agent 映射表:
  "ptm-tde" → consumer_status = "pending-consumer"
  "ptm-te"  → consumer_status = "pending-consumer"
  "ptm-tae" → consumer_status = "pending-consumer"
  "ptm-qa"  → consumer_status = "pending-consumer"
  其他      → consumer_status = "blocked"
```

首版简化：不检查目标 Agent 的安装/可用状态。所有合法 target_agent 设为 `pending-consumer`。

### 8.4 template 目录结构约定

```
skills/improvement-tracker/templates/
├── capa-proposal.yaml      # CA/PA 草案 schema + 示例
├── approved-input.yaml     # Approved Improvement Input schema + 示例
├── action-item.yaml        # ST-RA-04 负责
├── effectiveness-check.yaml # ST-RA-04 负责
├── closure-decision.yaml   # ST-RA-04 负责
└── measure-baseline.yaml   # ST-RA-06.3-TRACK 负责
```

本 Story 只创建前两个模板文件。

### 8.5 SKILL.md 写入策略

本 Story 在 `SKILL.md` 中写入 CA/PA 侧定义。写入结构：

```markdown
# improvement-tracker

## 1. 目标
...
## 2. 触发词与适用场景
...
## 3. CA/PA 治理（§3 本 Story 写入）
### 3.1 CA/PA 草案生成
### 3.2 批准状态机
### 3.3 Approved Improvement Input 生成
### 3.4 消费者映射
## 4. 闭环跟踪（§4 由 ST-RA-04 追加）
## 5. 措施基线管理（§5 由 ST-RA-06.3-TRACK 追加）
```

使用显式分节标记确保后续 Story 可安全追加。

---

## 9. 安全与性能设计

### 9.1 安全设计

| 安全约束 | 实现方式 | 验证方式 |
|---|---|---|
| 未批准不分发 | proposal approval_status gate | fixture：未批准 proposal → generate input 拒绝 |
| 不可自动批准 | 状态机规则：approval_status 变更仅由 reviewer 触发 | fixture：模拟 AI 自动批准 → 拒绝 |
| 不可自动分发到下游 | 不调用下游 Skill 工具，只写入文件 | fixture：检查输出不触发下游写入 |
| Approved Input 不可变 | immutable 标记 + 修改拒绝规则 | fixture：修改已生成 input → 拒绝 |
| 不读取凭据 | Skill 不声明凭据相关工具 | 工具声明审查 |
| 不进行外部读写 | Skill 不声明网络/外部 API 工具 | 工具声明审查 |

### 9.2 权限模型

| 角色 | 权限 |
|---|---|
| 测试架构师（ptm-tse） | 调用 Skill 生成 CA/PA 草案 |
| 人工 Reviewer | 批准/拒绝 CA/PA；查看所有 draft/approved/rejected proposals |
| 下游 Agent（ptm-tde/te/tae/qa） | 只读已批准 Approved Input |
| AI/Skill | 草案生成、schema 校验、门控规则执行；禁止改变 approval_status、禁止生成未批准 Input |

### 9.3 性能考虑

本 Story 涉及的对象均为文件化产物，数量级小（每 RA 报告 1-N 个 CA/PA）。不存在性能瓶颈场景。文件大小估计：
- `capa-proposal.yaml`：约 0.5-1 KB/项
- `approved-input.yaml`：约 0.3-0.5 KB/项

---

## 10. 测试设计

### 10.1 测试策略

| 测试类型 | 覆盖范围 | 验证方式 |
|---|---|---|
| Schema 校验 | CA/PA Proposal 必填字段、Approved Input 必填字段 | 静态 YAML schema 检查 |
| 状态机测试 | draft → approved/rejected 路径、不可回退 | approach fixture |
| 门控测试 | 未确认 RA 拒绝生成、未批准拒绝生成 Input | negative fixture |
| 不可变性测试 | Approved Input 生成后不可修改 | boundary fixture |
| 消费者映射测试 | target_agent 合法/非法值 | approach + boundary fixture |

### 10.2 测试场景清单

| ID | 场景 | 类型 | 输入 | 预期输出 |
|---|---|---|---|---|
| T-RA-03-SCHEMA-01 | CA/PA Proposal 必填字段完备 | schema | 包含所有必填字段的 proposal 草案 | 通过 schema 校验 |
| T-RA-03-SCHEMA-02 | CA/PA Proposal 缺少必填字段 | schema | 缺少 basis 的 proposal 草案 | schema 校验失败，提示缺失字段 |
| T-RA-03-SCHEMA-03 | Approved Input 必填字段完备 | schema | 包含所有必填字段的 input | 通过 schema 校验 |
| T-RA-03-GATE-01 | 未确认 RA 拒绝生成 CA/PA | negative | analysis_status != "analysis-confirmed" 的 RA Report | 拒绝，提示"分析结论未确认" |
| T-RA-03-GATE-02 | 已确认 RA 生成 CA/PA 草案 | approach | analysis_status = "analysis-confirmed" | 生成 draft proposal，含 7 个必填字段 |
| T-RA-03-GATE-03 | 未批准 proposal 拒绝生成 Approved Input | negative | approval_status = "draft" 的 proposal | 拒绝，提示"CA/PA 未批准" |
| T-RA-03-GATE-04 | 已拒绝 proposal 拒绝生成 Approved Input | negative | approval_status = "rejected" 的 proposal | 拒绝，保留 rejected 记录 |
| T-RA-03-FSM-01 | reviewer 批准 CA/PA | approach | draft proposal + reviewer approve | approval_status = "approved", approval_ref 填充 |
| T-RA-03-FSM-02 | reviewer 拒绝 CA/PA | approach | draft proposal + reviewer reject | approval_status = "rejected", comments 含拒绝原因 |
| T-RA-03-FSM-03 | 不可回退已批准状态 | boundary | approved proposal + 尝试改回 draft | 拒绝状态变更 |
| T-RA-03-FSM-04 | 不可回退已拒绝状态 | boundary | rejected proposal + 尝试改回 draft | 拒绝状态变更 |
| T-RA-03-IMMUT-01 | Approved Input 生成后不可修改 | boundary | 已生成 Approved Input + 尝试修改 scope | 拒绝修改 |
| T-RA-03-CONS-01 | 合法 target_agent → pending-consumer | approach | target_agent = "ptm-tde" | consumer_status = "pending-consumer" |
| T-RA-03-CONS-02 | 非法 target_agent → blocked | boundary | target_agent = "unknown-agent" | consumer_status = "blocked" |
| T-RA-03-CONS-03 | Approved Input 含所有追溯字段 | approach | 批准后生成 | source_ra + proposal_id + approval_ref 全部非空 |

### 10.3 与 Feature TEST-PLAN 的对齐

| Feature TEST-PLAN ID | 本 Story 测试覆盖 | 映射 |
|---|---|---|
| T-IMP-01 | T-RA-03-GATE-02 | 基于已确认 RA 生成 CA/PA 草案 |
| T-IMP-02 | T-RA-03-GATE-01 | 未确认 RA 不能生成 CA/PA |
| T-IMP-03 | T-RA-03-FSM-01 | reviewer 批准 CA/PA |
| T-IMP-04 | T-RA-03-FSM-02 | reviewer 拒绝 CA/PA |
| T-IMP-05 | T-RA-03-CONS-03 | 批准后生成 Approved Input |
| T-IMP-06 | T-RA-03-GATE-03/04 | 未批准不生成 Approved Input |
| T-IMP-07 | T-RA-03-IMMUT-01 | Approved Input 不可变性 |
| T-IMP-08 | T-RA-03-CONS-02 | 消费者不可用标为 blocked |
| T-IMP-09 | T-RA-03-SCHEMA-01/02 | CA/PA Proposal 字段完整性 |

---

## 11. 实施步骤

| 步骤 | 任务 | 输出 | 依赖 |
|---|---|---|---|
| 1 | 创建 `skills/improvement-tracker/` 目录结构 | 目录 + SKILL.md 框架 | 无 |
| 2 | 在 SKILL.md 写入 §3 CA/PA 治理定义（触发词、前置条件、流程） | SKILL.md CA/PA 侧 | 步骤 1 |
| 3 | 创建 `templates/capa-proposal.yaml`（schema + 示例 + 必填字段说明） | capa-proposal.yaml | 步骤 1 |
| 4 | 实现 CA/PA 草案生成逻辑（SKILL.md §3.1） | SKILL.md 流程定义 | 步骤 2、3 |
| 5 | 实现批准状态机（SKILL.md §3.2）：draft→approved/rejected，不可回退 | SKILL.md 状态机 | 步骤 2 |
| 6 | 创建 `templates/approved-input.yaml`（schema + 示例 + 不可变标记） | approved-input.yaml | 步骤 1 |
| 7 | 实现 Approved Input 生成逻辑（SKILL.md §3.3）：门控 + 生成 + 不可变 | SKILL.md 流程定义 | 步骤 4、5、6 |
| 8 | 实现消费者映射逻辑（SKILL.md §3.4）：target_agent → consumer_status | SKILL.md 映射表 | 步骤 7 |
| 9 | 创建 Fixture 文件（已确认 RA、draft/rejected/approved proposal、Approved Input） | Fixture 数据 | 步骤 3-8 |
| 10 | 本地静态验证：schema 校验、字段完整性、门控规则 | 验证记录 | 全部 |

---

## 12. 风险、难点与预研建议

### 12.1 风险

| 风险 ID | 描述 | 概率 | 影响 | 缓解措施 |
|---|---|---|---|---|
| R-LLD-RA-03-01 | improvement-tracker SKILL.md 由三个 Story 串行写入，写入冲突或覆盖风险 | 中 | 高 | 使用显式分节标记（§3/§4/§5），严格串行顺序，ST-RA-03→ST-RA-04→ST-RA-06.3-TRACK |
| R-LLD-RA-03-02 | 首版未实现下游就绪检测，`pending-consumer` 可能误导下游以为立即可消费 | 中 | 低 | 文档中注明"首版不检测下游就绪"，CP7 fixture 中 consumer 不可用不会改变 producer 状态 |
| R-LLD-RA-03-03 | CA/PA Proposal 的 basis 字段可能引用到分析结论中未确认的部分 | 低 | 中 | 生成逻辑中要求 RA Report 的 analysis_status=analysis-confirmed |

### 12.2 难点

1. **三 Story 串行共享 SKILL.md 的写入协调**：ST-RA-03（CA/PA 侧）、ST-RA-04（跟踪侧）、ST-RA-06.3-TRACK（基线侧）需协调 SKILL.md 的结构。方案：使用固定章节锚点，每个 Story 在独立章节内写入，不跨章节修改。

2. **Approved Input 不可变性如何实现**：由于基于文件系统，无法通过文件系统权限实现真正不可变。方案：Skill 规则中声明"生成后拒绝修改"，由 CP7 fixture 验证。

### 12.3 预研建议

- 无需要预研的未知项。CA/PA Proposal 和 Approved Input 的 schema 已在 Feature DESIGN 中明确。

### 开放项

| ID | 主题 | 状态 | 影响 | 重访条件 |
|---|---|---|---|---|
| OPEN-RA-03-01 | 下游消费完成后如何回写 consumer_status | OPEN | ST-RA-04 需要从 improvement-tracker 读取 consumer_status | CP6 实现时与 ST-RA-04 协调 |
| OPEN-RA-03-02 | CA/PA Proposal 文件存储路径约定 | OPEN | 未来跨 Story 引用时需要统一路径 | CP5 Decision Brief 确认 |

---

## 13. 回滚与发布策略

### 13.1 回滚策略

| 回滚场景 | 回滚操作 | 影响范围 |
|---|---|---|
| SKILL.md CA/PA 侧逻辑错误 | 回退到上一版本 SKILL.md（git revert） | improvement-tracker Skill |
| capa-proposal.yaml 或 approved-input.yaml schema 错误 | 修正模板文件 | 仅影响新生成的 proposal/input |
| 已生成 Approved Input 有误 | 由 reviewer 创建更正 proposal → 批准 → 生成新版本 Input | 旧 Input 保留，不删除 |
| 需要完全下线 CA/PA 侧 | 删除 SKILL.md CA/PA 侧内容，或标记为 deprecated | improvement-tracker Skill 降级 |

### 13.2 发布策略

本 Story 随 CR-030 Wave 3 一起发布，与 ST-RA-04 和 ST-RA-06.3-TRACK 共同构成完整的 improvement-tracker Skill。不能独立交付。

---

## 14. Definition of Done

- [ ] `skills/improvement-tracker/SKILL.md` 存在，CA/PA 侧（§3）完整
  - [ ] §3.1 CA/PA 草案生成流程定义
  - [ ] §3.2 批准状态机（draft → approved/rejected）
  - [ ] §3.3 Approved Input 生成逻辑
  - [ ] §3.4 消费者映射逻辑
- [ ] `skills/improvement-tracker/templates/capa-proposal.yaml` 存在，含完整 schema + 示例
- [ ] `skills/improvement-tracker/templates/approved-input.yaml` 存在，含完整 schema + 示例
- [ ] 未确认 RA → 拒绝生成 CA/PA（fixture 验证）
- [ ] 未批准 proposal → 拒绝生成 Approved Input（fixture 验证）
- [ ] 批准状态机不可回退（fixture 验证）
- [ ] Approved Input 生成后不可修改（fixture 验证）
- [ ] 消费者映射正确（合法 target_agent → pending-consumer，非法 → blocked）
- [ ] CA/PA Proposal 和 Approved Input 模板通过 YAML schema 校验
- [ ] 与 ST-RA-04 写入 SKILL.md 无冲突（独立节 §3/§4）
- [ ] DEV-LOG.md 追加记录

## 15. 实现灰区与取舍记录

| 灰区 ID | 问题 | 决策 | 理由 | 影响面 |
|---|---|---|---|---|
| GY-RA-03-01 | downstream readiness check 是否本期实现 | 否，首版不检测下游就绪 | 当前无 runtime 能力，检测下游需要 platform adapter | consumer_status=blocked 仅因 target_agent 非法触发 |
| GY-RA-03-02 | CA/PA Proposal 的 rejected 状态后能否重新提交 | 不可以，需创建新 proposal | rejected 是终态，保留审计记录 | 用户需重新生成 proposal 并走批准流程 |
| GY-RA-03-03 | Approved Input 的文件存储位置 | 与 SKILL.md 同目录的 `outputs/` 子目录 | 避免与模板混淆，统一输出路径 | 所有 Story 引用时需使用统一路径 |

---

## 16. 与相邻模块的集成契约

### 16.1 上游（reverse-analysis）

ST-RA-03 依赖 ST-RA-05.3-ANALYZE 产出。RA Report 必须：
- `analysis_status = "analysis-confirmed"`
- `confirmed_by` 非空
- `confirmed_at` 非空
- 包含改进维度的分析结论

不满足时 ST-RA-03 的 CA/PA 草案生成器拒绝执行。

### 16.2 下游（ST-RA-04 闭环跟踪）

ST-RA-04 依赖 ST-RA-03 产出的 Approved Improvement Input 作为跟踪基础。Approved Input 的 `proposal_id` 和 `input_id` 将被 ST-RA-04 的 Action Item 引用。

### 16.3 下游（ST-RA-06.3-TRACK 措施基线）

ST-RA-06.3-TRACK 的 MeasureBaseline 通过 `proposal_ref` 引用 ST-RA-03 产出的 CA/PA Proposal。

### 16.4 跨 Agent 消费者（ptm-tde/te/tae/qa）

Approved Improvement Input 是唯一跨 Agent 流动对象。消费方式为只读文件读取。消费者不得修改 Input 内容。消费者通过 `consumer_status` 了解 Input 状态。

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-16 | host-orchestrator | CP5 B12：迁移为当前 full-lld 证据兼容格式，增加 canonical Story ID 与 §0 工程依据/章节索引；不改变原设计契约。 |
