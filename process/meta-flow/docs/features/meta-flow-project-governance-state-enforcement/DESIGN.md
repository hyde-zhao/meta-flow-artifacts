---
title: "Feature Design: Meta Flow Project Governance and State Enforcement"
status: "baseline"
created_at: "2026-07-02"
owner: "host-orchestrator"
cr_ref: "CR-037"
---

# Feature Design: Meta Flow Project Governance and State Enforcement

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v1.0 | 2026-07-02 | host-orchestrator | 建立 Feature 级设计基线。 |

## Feature 目标

本 Feature 将 CR-037 的治理整改拆成可实现、可测试、可迁移的工程切片，优先修复 current state enforcement，再补 project governance 和 roadmap refresh。

## 证据层级说明

本文档是 CR-037 的 umbrella 设计基线，用于承接已评审实施计划和产品 / 架构基线。它不是 FEAT-PG-001..009 的 per-feature CP5 设计证据，也不能单独授权进入实现。

后续在 CP3 通过后、CP5 前，必须按 `process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md` 生成各 Feature 的独立 `DESIGN.md`、`TEST-PLAN.md` 和 `TASKS.md`，并让 Story 的 `feature_design_refs` 指向对应 per-feature 目录。

## 实现对象

| 对象 | 预计文件 | 说明 |
|---|---|---|
| State allowlist | `meta_flow/state/current.py` | allowlist、budget、audit/enforce |
| State update API | `meta_flow/state/current.py` | deep-merge patch，统一写入入口 |
| CR lifecycle 收敛 | `meta_flow/workflow/cr_lifecycle.py` | 替换直接 `write_text()` |
| Workspace scaffold | `meta_flow/workspace/routing.py` | 增加 `process/project` |
| Project state schema | 新增 `meta_flow/project/*` 或等价模块 | `PROJECT.current.json`、project files |
| Impact normalization | `meta_flow/workflow/cr_lifecycle.py`、`meta_flow/checks/*` | surface/path/ref 分离 |
| Registry refs checker | `meta_flow/design/product_governance.py` 或新增 checker | feature/capability refs 校验 |
| Roadmap refresh | 新增 project/roadmap checker | result schema、stale report、GATE-LEDGER event |
| FU-RF support | `meta_flow/checks/cr_tracking.py`、模板 | 扩展候选 ID |
| Contracts/docs | delivery rules、skills、README | 写契约同步 |

## 数据结构草案

### Current State Allowlist

```yaml
allowed_current_keys:
  required:
    - schema_version
    - project_id
    - workflow_mode
    - current_phase
    - blocked
    - active_change
    - active_story
    - pending_gate
    - next_action
    - routing_ref
    - active_context_ref
    - authz_policy_refs
    - open_risks
    - updated_at
    - source_refs
  optional:
    - pending_checklist_path
    - project_state_ref
```

### Project Current

```yaml
schema_version: 1
project_id: meta-flow
project_positioning: "meta workflow factory"
project_scale: full
current_project_phase: "governance-hardening"
active_objective_refs: []
active_milestone_refs: []
roadmap_ref: process/project/ROADMAP.yaml
milestone_ref: process/project/MILESTONES.yaml
health_ref: process/project/PROJECT-HEALTH.summary.json
updated_at: ""
```

### Roadmap Refresh Result

```yaml
schema_version: 1
result_type: roadmap_refresh
refresh_id: RF001
decision: UPDATED_WITH_DOC_IMPACTS
machine_updates: []
must_check: []
stale_items: []
follow_up_candidates: []
event_refs: []
```

## 失败处理

| 失败 | 行为 |
|---|---|
| unknown current state key | audit WARN / enforce ERROR |
| budget 超限 | ERROR |
| capability id 未注册 | blocked finding + FU-RF |
| roadmap refresh 需要改发布库 | 只生成 stale item / FU-RF |
| active CR 冲突 | 不更新 active state，等待用户授权 |

## Gotchas

- 不要为了迁移方便把 legacy 字段加入 allowlist；必须判断字段是否属于短状态。
- 不要把 `next_session_handoff_ref` 这类偶发字段直接合法化；优先迁移到 handoff/ledger。
- 不要让 project state 成为新的巨型状态文件。
- 不要让 migration 自动创造 quant-lab capability id。
