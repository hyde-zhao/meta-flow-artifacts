---
checkpoint_id: CP2
checkpoint_name: REQUIREMENTS-BASELINE
type: auto_precheck
phase: requirement-clarification
status: PASS
created_at: 2026-05-18
owner: meta-pm
manual_checkpoint: checkpoints/CP2-REQUIREMENTS-BASELINE.md
target:
  phase: requirement-clarification
  artifacts:
    - process/USE-CASES.md
    - process/REQUIREMENTS.md
    - process/changes/CR-001.md
    - README.md
source_documents:
  - process/USE-CASES.md
  - process/REQUIREMENTS.md
  - process/changes/CR-001.md
  - README.md
human_checkpoint: checkpoints/CP2-REQUIREMENTS-BASELINE.md
---

# CP2 Requirements Baseline

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP0 自动检查完成 | PASS | `process/checks/CP0-REQUEST-INTAKE.md`。 |
| CP1 自动检查完成 | PASS | `process/checks/CP1-USE-CASE-COMPLETENESS.md`。 |
| 需求文档存在 | PASS | `process/REQUIREMENTS.md` version `1.4`。 |
| 需求来源明确 | PASS | `source_use_cases` 为 `UC-05..UC-10`。 |
| 变更单已闭环 | PASS | `process/changes/CR-001.md` 已为 `status: closed`，关闭依据为 CP2 人工确认。 |

## Checklist

| 检查项 | 结果 | 说明 |
|---|---|---|
| 需求状态正确 | PASS | `REQUIREMENTS.md` 为 `status: confirmed`，`ready_for_design: true`，符合 CP2 通过后状态。 |
| 功能需求覆盖 | PASS | `R-F-012..R-F-021` 覆盖安装、初始化、登录、登录态守卫、配置执行、验证诊断与设计输入缺口。 |
| 约束需求覆盖 | PASS | `R-C-008..R-C-014` 覆盖 `.input` 边界、敏感信息、登录状态、阶段门控、自动回滚排除和 README 事实。 |
| 非功能需求覆盖 | PASS | `R-NF-006..R-NF-010` 覆盖追溯、可验证性、边界清晰、README 一致性和确认门控。 |
| 验收条件可检查 | PASS | 每条需求包含可独立审查的验收条件，主要采用 Given/When/Then。 |
| README 一致性 | PASS | 需求不假定当前 CLI 已具备真实设备执行能力；后续交付面仍由 README 原生目录承载。 |
| 旧基线追溯 | PASS | `REQUIREMENTS.md` 保留旧需求到当前需求对象的映射。 |
| 敏感信息边界 | PASS_WITH_NOTE | 唯一允许显式出现的密码值为用户指定目标密码策略 `Ngfw@123`；未发现真实设备 IP、token、cookie 或 FTP 凭据值。 |
| 开放问题处理 | PASS | CP2 已确认默认处理：`OI-001` 不扩边，`OI-002` 与 `OI-003` 延后 HLD，`OI-004` 排除出当前主基线。 |
| 阶段门控 | PASS | CP2 已批准；当前可进入 `solution-design`，但不得跳过 HLD 评审。 |

## Open Items For Human Review

| ID | 当前建议 | 说明 |
|---|---|---|
| OI-001 | 默认不扩展，除非用户修改 | 初始化保持关闭验证码、配置管理路由、统一改密；启用 SSH、恢复 license 暂不纳入主基线。 |
| OI-002 | 延后到 HLD | 登录状态记录采用内存态、文件态还是执行上下文引用，需要设计阶段给出不含敏感载荷的状态模型。 |
| OI-003 | 延后到 HLD | 配置后验证是否按 capacity 接口族细分，由 HLD 比较统一验证与细粒度验证方案。 |
| OI-004 | 暂不纳入 | 自动回滚仅在用户后续明确要求时通过新 CR 引入。 |

## Exit Criteria

| 条件 | 自动检查状态 | 人工状态 |
|---|---|---|
| 场景与需求一致 | PASS | APPROVED |
| 需求可追溯且可验证 | PASS | APPROVED |
| 高风险边界已显式记录 | PASS | APPROVED |
| `ready_for_design` 可置为 `true` | PASS | APPROVED |
| 可进入 `solution-design` | PASS | APPROVED |

## Deliverables

| 文件 | 状态 | 说明 |
|---|---|---|
| `process/checks/CP2-REQUIREMENTS-BASELINE.md` | CREATED | 自动预检结果。 |
| `checkpoints/CP2-REQUIREMENTS-BASELINE.md` | APPROVED | 人工确认稿已回填用户“通过”。 |
| `process/USE-CASES.md` | CONFIRMED | 已确认。 |
| `process/REQUIREMENTS.md` | CONFIRMED | `ready_for_design: true`，可进入 HLD。 |
| `process/changes/CR-001.md` | CLOSED | 需求更新已完成并关闭。 |

## Decision

CP2 自动预检通过，人工确认已通过。当前阶段可进入 `solution-design`，下一步是唤醒 `meta-se` 输出 HLD 与 ADR。
