---
status: draft
version: "1.0"
source_cr: "CR-030"
source_hld: "docs/design/HLD.md (v1.2, REV-03)"
feature_id: "FEAT-RA-IMPROVEMENT"
agent: "ptm-tse"
created_by: "meta-se"
created_at: "2026-07-16"
---

# FEAT-RA-IMPROVEMENT: 改进输入治理 — Feature 设计

> 对应 HLD REV-03 的 `improvement-tracker` Skill CA/PA 侧。

## 1. Feature 概述

### 目标

基于已确认的分析结论，生成可审计的 CA/PA 草案，经人工 reviewer 批准后产出不可变的 Approved Improvement Input，供下游 Agent 只读消费。

### 成功标准

| 指标 | 度量方式 | 目标值 |
|---|---|---|
| 未批准不分发 | 未批准 CA/PA 的 Approved Input 数为 0 | fixture 证明 |
| 输入可追溯 | 100% Approved Input 含 source_ra、proposal_id、approval_ref | schema check |
| CA/PA 完整性 | 每项含依据、类型、目标、Owner、验收、有效性检查 | 100% |

## 2. 关键接口

### 2.1 improvement-tracker Skill（CA/PA 侧）

| 属性 | 值 |
|---|---|
| 输入 | 已确认 RA Report（analysis-confirmed 状态） |
| 输出 | CA/PA Proposal（草案）+ Approved Improvement Input（批准后） |
| 状态管理 | draft → approved/rejected → pending-consumer → consumed/blocked |
| 禁止 | 未批准分发、自动建下游任务、修改分析结论 |

### 2.2 Approved Improvement Input Schema

```yaml
input_id: string
source_ra: string            # 来源 RA Report ID
proposal_id: string          # 关联 CA/PA Proposal ID
kind: enum                   # 'corrective' | 'preventive'
target_agent: string         # 目标消费 Agent（ptm-tde/ptm-te/ptm-tae/ptm-qa）
scope: string                # 适用范围
acceptance_criteria: string  # 验收标准
priority: enum               # 'P0' | 'P1' | 'P2'
constraints: string          # 限制条件
approval_ref:
  reviewer: string
  approved_at: datetime
consumer_status: enum        # 'pending-consumer' | 'consumed' | 'blocked'
```

### 2.3 CA/PA Proposal Schema

```yaml
proposal_id: string
analysis_ref: string         # RA Report + analysis_run 引用
kind: enum
title: string
basis: string                # 依据（分析结论引用）
target: string               # 改进目标
owner: string
due_date: date
validation_method: string    # 验证方式
side_effects: string         # 潜在副作用
approval_status: enum        # 'draft' | 'approved' | 'rejected'
approval_ref:
  reviewer: string
  approved_at: datetime
  comments: string
```

## 3. 状态机

```
draft ──(reviewer approve)──► approved ──► pending-consumer
  │                              │                │
  └──(reviewer reject)──► rejected         ┌────┘
                                            ▼
                                      consumed / blocked
```

- 批准后 CA/PA Proposal 不可变
- Approved Improvement Input 在批准后才创建
- consumer 不可用时标为 blocked，不自动重试

## 4. 关键流程

```
已确认 RA Report
    │
    ▼
生成 CA/PA 草案（依据 + 目标 + Owner + 验收）
    │
    ▼
人工 reviewer 审查 → 批准/拒绝
    │ (approved)
    ▼
生成 Approved Improvement Input（不可变）
    │
    ▼
映射到目标消费者 Agent
    │
    ▼
消费者读取（只读）→ 回链
```

## 5. Gotchas

- **不要把 CA/PA 草案直接交给下游**：必须经过批准门
- **不要自动批准**：只有人工 reviewer 能改变 approval_status
- **不要把 Approved Input 当作下游任务**：下游自行决定如何处理
- **不要让拒绝的 proposal 消失**：保留审计记录
- **不要假设消费者立即可用**：支持 pending-consumer/blocked 状态

---

# FEAT-RA-IMPROVEMENT — 测试计划

| ID | 场景 | Story | 预期 |
|---|---|---|---|
| T-IMP-01 | 基于已确认 RA 生成 CA/PA 草案 | ST-RA-03 | 每项含依据/类型/目标/Owner/验收 |
| T-IMP-02 | 未确认 RA 不能生成 CA/PA | ST-RA-03 | 拒绝，提示需先确认分析 |
| T-IMP-03 | reviewer 批准 CA/PA | ST-RA-03 | proposal.status='approved' |
| T-IMP-04 | reviewer 拒绝 CA/PA | ST-RA-03 | proposal.status='rejected'，保留记录 |
| T-IMP-05 | 批准后生成 Approved Improvement Input | ST-RA-03 | 输入含 source_ra/proposal_id/approval_ref |
| T-IMP-06 | 未批准不生成 Approved Input | ST-RA-03 | 0 个未批准输入 |
| T-IMP-07 | Approved Input 不可变性 | ST-RA-03 | 生成后字段不可修改 |
| T-IMP-08 | 消费者不可用标为 blocked | ST-RA-03 | consumer_status='blocked' |
| T-IMP-09 | CA/PA Proposal 字段完整性 | ST-RA-03 | 必填字段齐全 |

---

# FEAT-RA-IMPROVEMENT — 任务清单

### ST-RA-03: 改进输入治理

**TASK-IMP-01**: 在 `skills/improvement-tracker/SKILL.md` 写入 CA/PA 侧定义
**TASK-IMP-02**: 实现 CA/PA Proposal 草案生成（依据 + 目标 + Owner + 验收）
**TASK-IMP-03**: 实现批准状态机（draft→approved/rejected）
**TASK-IMP-04**: 实现 Approved Improvement Input 生成（批准的 CA/PA → 不可变输入）
**TASK-IMP-05**: 实现消费者映射逻辑（target_agent → consumer_status）
**TASK-IMP-06**: 实现未批准门控（未确认 RA / 未批准 CA/PA → 拒绝生成）
**TASK-IMP-07**: 创建 CA/PA Proposal 和 Approved Input 模板
**TASK-IMP-08**: 创建 Fixture（已确认 RA、CA/PA 草案、批准/拒绝场景）
