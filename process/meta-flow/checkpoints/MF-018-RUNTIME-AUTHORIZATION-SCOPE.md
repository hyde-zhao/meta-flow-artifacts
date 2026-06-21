---
checkpoint_id: "MF-018-RUNTIME-AUTHORIZATION-SCOPE"
checkpoint_name: "MF-018 Runtime Authorization Scope"
type: "manual"
status: "blocked"
owner: "host-orchestrator"
created_at: "2026-06-21T17:06:24+08:00"
reviewed_by: "user"
reviewed_at: "2026-06-21T17:06:24+08:00"
source_context: "process/context/MF-018.RUNTIME-AUTHORIZATION.context.yaml"
target:
  phase: "runtime-authorization-preparation"
  story_id: "MF-018"
  selected_option: "option_c_true_publish_or_production_write"
---

# MF-018 Runtime Authorization Scope

## 用户选择

用户回复 `c`，解释为选择 `option_c_true_publish_or_production_write`：真实发布 / live 执行 / production write 路径。

## 当前结论

| 字段 | 内容 |
|---|---|
| Scope 选择 | `option_c_true_publish_or_production_write` |
| 执行状态 | `not-started` |
| 授权状态 | `staged-component-validation-planned`，C0 仍 `blocked` |
| 阻断原因 | 高风险 runtime 操作缺少完整授权输入，且真实 runtime 前必须完成组件验证 |
| 下一步 | 先完成 C0 Runtime target definition，再按 C1-C8 分阶段推进 |

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP8 已批准 | PASS | `process/checkpoints/CP8-MF-018-DELIVERY-READINESS.md` | MF-018 delivery readiness approved |
| Runtime handoff 已创建 | PASS | `process/handoffs/MF-018-RUNTIME-AUTHORIZATION-HANDOFF.md` | handoff-only |
| 用户选择 scope | PASS | 当前对话：`c` | 选择 C，高风险路径 |
| 高风险授权输入完整 | BLOCKED | 本文件 `## 缺失输入` | 缺少目标环境、写入范围、凭据边界、回滚等 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | 是否包含真实发布 / production write | PASS | 用户选择 C | 必须走 high-risk gate |
| 2 | Dry-run / sandbox 是否优先 | FAIL | 用户选择 C | 建议先降级到 A/B；若坚持 C，必须补齐完整授权输入 |
| 3 | 权限最小化 | BLOCKED | 缺少目标和写入范围 | 无法判定 |
| 4 | 超时保护 | BLOCKED | 缺少命令 / endpoint | 必须提供 `timeout_seconds` |
| 5 | 失败保护 | BLOCKED | 缺少回滚和 on_failure | 必须提供 abort / rollback / retry limit |
| 6 | 凭据边界 | BLOCKED | 缺少 secret policy | 不得读取或记录任何凭据 |
| 7 | 证据脱敏 | BLOCKED | 缺少 redaction policy | 外部输出写入 evidence 前必须脱敏 |

## 缺失输入

| 输入项 | 当前状态 | 必填说明 |
|---|---|---|
| 具体操作 | missing | publish / live release / production write / artifact deletion / other |
| 目标环境 | missing | provider、account/project、sandbox/staging/production、endpoint 或命令 |
| 写入范围 | missing | 允许改变的文件、记录、仓库、服务或外部对象 |
| 凭据边界 | missing | secret 名称或来源策略、是否允许 Codex 访问、脱敏要求 |
| 网络授权 | missing | allowed hosts、协议、timeout_seconds |
| 回滚负责人和步骤 | missing | rollback owner、rollback command/procedure |
| blast radius | missing | 最大允许变更数量、停机窗口、停止阈值 |
| on_failure | missing | abort / rollback / retry limit / evidence capture |
| 成功标准 | missing | 可验证的成功条件和观察窗口 |
| 执行授权原文 | missing | precheck 通过后仍需单独明确授权执行 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 可进入 precheck | BLOCKED | 缺失输入未补齐 | 不能继续到执行；只能先进入 C0 target definition |
| 可执行外部操作 | BLOCKED | 未授权 | 当前不得执行任何外部 runtime 操作 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Runtime context | `process/context/MF-018.RUNTIME-AUTHORIZATION.context.yaml` | updated | 记录 C 已选择 |
| Runtime handoff | `process/handoffs/MF-018-RUNTIME-AUTHORIZATION-HANDOFF.md` | available | handoff-only |
| Scope checkpoint | `process/checkpoints/MF-018-RUNTIME-AUTHORIZATION-SCOPE.md` | BLOCKED | 等待补齐高风险输入 |
| Component validation plan | `process/checks/MF-018-RUNTIME-COMPONENT-VALIDATION-PLAN.md` | planned | C0-C8 分阶段完成真实 runtime |

## 人工审查结果

- 结论：`blocked`
- 审查人：user
- 审查时间：2026-06-21T17:06:24+08:00
- 修改意见：用户选择 C，但缺少高风险执行必填输入；当前不得执行。
- 风险接受项：未形成执行风险接受；需另行 runtime-high-risk-human-gate。
