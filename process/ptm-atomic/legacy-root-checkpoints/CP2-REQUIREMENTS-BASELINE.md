---
checkpoint_id: CP2
checkpoint_name: REQUIREMENTS-BASELINE
type: auto_then_manual
phase: requirement-clarification
status: approved
created_at: 2026-05-18
owner: meta-po
auto_check_result: process/checks/CP2-REQUIREMENTS-BASELINE.md
target:
  phase: requirement-clarification
  artifacts:
    - process/USE-CASES.md
    - process/REQUIREMENTS.md
    - process/changes/CR-001.md
source_documents:
  - process/USE-CASES.md
  - process/REQUIREMENTS.md
  - process/checks/CP2-REQUIREMENTS-BASELINE.md
  - process/changes/CR-001.md
---

# CP2 Requirements Baseline Review

## Review Scope

本检查点用于人工确认 `CR-001` 后的场景与需求基线是否可以进入 HLD。确认前，流程保持在 `requirement-clarification`，`process/REQUIREMENTS.md` 的 `ready_for_design` 保持 `false`。

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP0 已完成 | PASS | `process/checks/CP0-REQUEST-INTAKE.md`。 |
| CP1 已完成 | PASS | `process/checks/CP1-USE-CASE-COMPLETENESS.md`。 |
| CP2 自动预检已完成 | PASS | `process/checks/CP2-REQUIREMENTS-BASELINE.md`。 |
| 场景文档可审阅 | PASS | `process/USE-CASES.md` version `1.4`。 |
| 需求文档可审阅 | PASS | `process/REQUIREMENTS.md` version `1.4`。 |

## Checklist

| 审查项 | 建议结果 | 审查说明 |
|---|---|---|
| `UC-05..UC-10` 是否正确表达当前业务主链路 | APPROVED | 覆盖防火墙安装、初始化、目标密码登录、登录态守卫、capacity 配置、后置健康检查/诊断。 |
| `R-F-012..R-F-021` 是否足以进入 HLD | APPROVED | 功能需求已覆盖主链路和 schema/CLI 支持缺口。 |
| 高风险边界是否可接受 | APPROVED | 需求已约束 `.input` 只读参考、敏感信息不落盘、真实设备执行能力不作为当前事实。 |
| 初始化范围是否接受当前最小集合 | APPROVED | 默认只包含串口登录、关闭验证码、配置管理路由、统一改密为 `Ngfw@123`。 |
| 登录状态持久化边界是否可延后到 HLD | APPROVED | 需求只定义“不保存原始密码、完整 token 和敏感会话载荷”，具体模型留给 HLD。 |
| 配置后验证粒度是否可延后到 HLD | APPROVED | 需求覆盖接口族来源，验证是否细分由 HLD 设计。 |
| 自动回滚是否继续排除 | APPROVED | 当前主基线只要求诊断与人工处理信号，不包含自动回滚。 |

## Open Items

| ID | 建议处理 | 用户确认结果 |
|---|---|---|
| OI-001：是否追加启用 SSH、恢复 license 等初始化动作 | 默认不追加；如需追加，后续通过新 CR 明确说明 | APPROVED_AS_DEFAULT |
| OI-002：登录状态记录持久化边界 | 延后到 HLD 设计，但必须不落敏感载荷 | DEFER_TO_HLD |
| OI-003：配置后验证是否按接口族细分 | 延后到 HLD 比较统一验证和细粒度验证方案 | DEFER_TO_HLD |
| OI-004：自动回滚是否进入主基线 | 当前不纳入；后续如需要通过新 CR 引入 | EXCLUDED_FROM_BASELINE |

## Approval Options

请只回复以下三种之一：

- `approve`
- `修改: <具体修改点>`
- `reject`

## Exit Criteria

| 用户回复 | 后续动作 |
|---|---|
| `approve` | 标记 `USE-CASES.md` 和 `REQUIREMENTS.md` 已确认；将 `ready_for_design` 改为 `true`；关闭 `CR-001`；更新 `STATE.md` 到 `solution-design`；随后唤醒 `meta-se`。 |
| `修改: <具体修改点>` | 保持 `requirement-clarification`；由 meta-po 组织需求修订，更新 CP1/CP2 记录后再次发起确认。 |
| `reject` | 保持 `requirement-clarification`；标记当前 CP2 不通过，并等待用户给出新的范围或回退决策。 |

## Human Review Result

| 字段 | 值 |
|---|---|
| result | APPROVED |
| reviewed_by | user |
| reviewed_at | 2026-05-18 |
| decision_notes | 用户回复“通过”；接受当前 `UC-05..UC-10` 与 `R-F-012..R-F-021` 基线，允许进入 HLD。 |
