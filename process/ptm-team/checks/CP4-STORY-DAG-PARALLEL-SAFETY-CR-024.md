---
check_id: CP4-STORY-DAG-PARALLEL-SAFETY-CR-024
workflow_id: WF-PTM-TEAM-20260520-001
change_id: CR-024-ptm-te-agent
cp: CP4
type: auto_precheck
status: PASS
checked_at: "2026-07-10T00:00:00+08:00"
checked_by: host-orchestrator（主进程）
development_plan: process/DEVELOPMENT-PLAN-CR-024.yaml
feature_design_matrix: docs/design/FEATURE-DESIGN-MATRIX.md
---

# CP4 - Story 拆解与并行安全门预检（CR-024）

## Entry Criteria

| 条件 | 状态 | 证据 |
|------|------|------|
| CP3 已 approved | ✓ | `process/checkpoints/CP3-HLD-REVIEW-CR-024.md`（approved 2026-07-10） |
| FEATURE-DESIGN-MATRIX 已生成 | ✓ | `docs/design/FEATURE-DESIGN-MATRIX.md`（F-011/F-012 -> CR-024 in-progress） |
| DEVELOPMENT-PLAN.yaml 已生成 | ✓ | `process/DEVELOPMENT-PLAN-CR-024.yaml`（S1-S4, 3 Wave） |
| HLD 锁定 Story 文件布局 | ✓ | HLD v1.1 §12 |

## Checklist

| # | 检查项 | 结果 | 说明 |
|---|--------|------|------|
| 1 | Story 拆解完整（S1-S4） | PASS | 4 Story，覆盖 HLD §12 全部文件布局 |
| 2 | 依赖关系无循环 | PASS | S1 <- S2,S3 <- S4；W1->W2->W3 线性 + W2 内并行 |
| 3 | lld_policy 分配（FEATURE-DESIGN-MATRIX） | PASS | S1/S2/S3 full-lld，S4 technical-note（HLD §12 + FEATURE-DESIGN-MATRIX 一致） |
| 4 | 文件所有权无冲突 | PASS | 4 Story 文件所有权互不重叠（见下表） |
| 5 | Wave 并行安全（W2） | PASS | S2/S3 不同 skill 目录，无文件重叠，可并行 |
| 6 | 跨 Wave 冲突检查 | PASS | S4（W3）文件与前 3 个 Story 无重叠 |
| 7 | Story tier 分配合理 | PASS | S1/S4 tier S（骨架/集成），S2/S3 tier M（实现复杂） |

## Story DAG

```
W1: STORY-024-01 (agent 骨架，full-lld)
      ↓
W2: STORY-024-02 (device mgmt+conn，full-lld)  ‖  STORY-024-03 (policy-route-exec，full-lld)
      ↓                                              ↓
W3:              STORY-024-04 (安装器集成，technical-note)
```

## 文件所有权冲突分析

| Story | Wave | 文件所有权 | 与其他 Story 重叠 |
|-------|:--:|-----------|----------------|
| STORY-024-01 | W1 | `agents/ptm-te.md` | 无 |
| STORY-024-02 | W2 | `skills/device-management/**`, `skills/device-connection/**` | 无 |
| STORY-024-03 | W2 | `skills/policy-route-execution/**` | 无（与 S2 不同 skill 目录） |
| STORY-024-04 | W3 | `script/ptm_team/install.py`, `skills/README.md`, `docs/ptm-team-blueprint.md` | 无（W3，前 3 个已完成） |

**W2 并行安全结论**：STORY-024-02（device-management + device-connection）与 STORY-024-03（policy-route-execution）文件所有权完全无重叠，可在 W2 并行实现。

**跨 Wave 安全结论**：STORY-024-04 修改的 `install.py` / `skills/README.md` / `blueprint.md` 不被 S1/S2/S3 修改，W3 实现时前 3 个 Story 已完成，无冲突。

## lld_policy 分配

| Story | lld_policy | 理由 |
|-------|------|------|
| STORY-024-01 | full-lld | agent 编排器核心，定义全部下游契约 + 三层映射 + 执行门控 |
| STORY-024-02 | full-lld | 双 skill 协调 + 连接逻辑实现 + 快照采集 |
| STORY-024-03 | full-lld | op_mapper 双层映射核心复杂点（8 op_id + 7 op flag） |
| STORY-024-04 | technical-note | 安装器集成 + 文档回写，低复杂度 |

> 无 runtime/security/credential/production-write 高风险 Story 使用 batch-lld；S1/S2/S3 各有独立复杂度，使用独立 full-lld。

## Exit Criteria

| 条件 | 状态 |
|------|------|
| Story DAG 无循环 | ✓ |
| 文件所有权无冲突 | ✓ |
| Wave 并行安全 | ✓ |
| lld_policy 分配完整 | ✓ |
| 可进入 CP5 LLD 设计证据 | ✓ |

## 结论

**CP4 自动预检 PASS（7/7）**。Story DAG 无循环，文件所有权无冲突，W2 可并行。按 Post-approval 自动推进，CP4 PASS 后推进 CP5（S1-S4 LLD 设计证据产出 + CP5 自动预检），在 CP5 人工门禁停下。

## Deliverables

- `process/DEVELOPMENT-PLAN-CR-024.yaml`（Story 管理机器真相源）
- `docs/design/FEATURE-DESIGN-MATRIX.md`（F-011/F-012 更新 + Story 映射）
- 本预检文件

## Agent Dispatch Evidence

| 字段 | 值 |
|------|------|
| mode | inline-fallback（CP4 自动预检，Host Orchestrator 主进程） |
| reason | CP4 是 Story DAG 并行安全预检，基于 HLD v1.1 §12 + DEVELOPMENT-PLAN，无需子 agent |
