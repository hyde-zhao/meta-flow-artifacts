---
checkpoint_id: CP1
checkpoint_name: USE-CASE-COMPLETENESS
type: auto
phase: requirement-clarification
status: PASS
created_at: 2026-05-18
owner: meta-pm
target:
  phase: requirement-clarification
  artifacts:
    - process/USE-CASES.md
    - process/CLARIFICATION-LOG.md
    - process/changes/CR-001.md
source_documents:
  - process/USE-CASES.md
  - process/changes/CR-001.md
  - process/INPUT-INDEX.md
---

# CP1 Use Case Completeness

## Entry Criteria

| 条目 | 状态 | 证据 |
|---|---|---|
| CP0 已完成 | PASS | `process/checks/CP0-REQUEST-INTAKE.md`。 |
| 场景文档存在 | PASS | `process/USE-CASES.md` version `1.3`。 |
| 变更基线存在 | PASS | `process/changes/CR-001.md` 描述本轮场景纠偏。 |
| 参考输入索引存在 | PASS | `process/INPUT-INDEX.md` 覆盖 `.input/capacity/` 与 `.input/ngfw-install/`。 |

## Checklist

| 检查项 | 结果 | 说明 |
|---|---|---|
| 当前主基线明确 | PASS | `USE-CASES.md` 明确当前主基线为 `UC-05..UC-10`，旧 `UC-01..UC-04` 仅追溯保留。 |
| 用户给定链路覆盖 | PASS | 覆盖防火墙安装、安装后初始化、目标密码登录、登录态守卫、capacity 配置执行、后置健康检查与诊断。 |
| 人物画像覆盖 | PASS | 覆盖防火墙交付与测试工程师、自动化编排开发者。 |
| 成功指标量化 | PASS | `SM-01..SM-04` 均包含目标值，且可通过文档审查验证。 |
| 补充场景评估 | PASS | 安装后健康检查、配置后验证/诊断、登录态失效重登、凭据/状态记录边界已处理；自动回滚暂不纳入主基线。 |
| `.input/` 边界 | PASS | 明确禁止直接复制外部源码、环境文件、日志、缓存、IDE 配置或凭据。 |
| 敏感信息边界 | PASS_WITH_NOTE | 文档仅允许出现用户指定目标密码策略 `Ngfw@123`；未发现真实设备 IP、token、cookie 或 FTP 凭据值。 |
| 开放问题状态 | PASS_WITH_OPEN_ITEMS | `OQ-01..OQ-04` 已列出，需在 CP2 或 HLD 中关闭、延后或接受为风险。 |

## Exit Criteria

| 条件 | 状态 | 说明 |
|---|---|---|
| 场景覆盖安装到验证主链路 | PASS | `UC-05..UC-10` 完整覆盖。 |
| 场景与 CR-001 可追溯 | PASS | `USE-CASES.md` 保留旧基线映射。 |
| 可进入 CP2 需求基线审查 | PASS | 场景足以支撑 `REQUIREMENTS.md` 审查。 |
| 可进入 HLD | BLOCKED | 仍需 CP2 人工确认。 |

## Deliverables

| 文件 | 状态 | 说明 |
|---|---|---|
| `process/USE-CASES.md` | DRAFT_READY_FOR_REVIEW | 等待 CP2 人工确认时一并审阅。 |
| `process/changes/CR-001.md` | OPEN | 场景纠偏已写入，人工确认未关闭。 |
| `process/checks/CP1-USE-CASE-COMPLETENESS.md` | CREATED | 本自动检查点。 |

## Decision

CP1 自动检查通过，但仍有开放问题。开放问题不阻塞生成 CP2 人工确认稿，但在用户批准前不得进入 HLD。
