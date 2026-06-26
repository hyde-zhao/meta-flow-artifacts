---
handoff_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-NEXT-SESSION-HANDOFF-2026-06-25"
created_at: "2026-06-25T16:22:35+08:00"
from: "host-orchestrator"
to: "next-host-orchestrator"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
semantic: "context-reset"
status: "ready-for-non-trading-window-work"
dispatch:
  required: false
  semantic: "stage-dispatch"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "next-host-orchestrator"
  canonical_role: "host-orchestrator"
  codex_agent_name: ""
  reasoning_profile: "default"
  dispatch_trigger: "context-reset-before-session-clear"
  agent_path: ""
  tool_name: ""
  agent_id: ""
  agent_name: ""
  thread_id: ""
  spawned_at: ""
  resumed_at: ""
  completed_at: ""
  evidence: "handoff-only; no subagent dispatch required"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Only if a required non-trading task lacks recoverable local context."
  forbidden_question_scope: "runtime authorization, credentials, raw account/order/funding data, publish, NAS, small_live/live, trading authorization"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-NEXT-SESSION-HANDOFF-2026-06-25.md"
  context_ref: "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-NEXT-SESSION-HANDOFF-2026-06-25.md"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 12
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "Session reset handoff must correct stale session_expired state and identify next non-trading-window tasks."
  allowed_reads:
    - "process/STATE.md"
    - "process/state/STATE.current.json"
    - "process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-VERIFICATION-2026-06-25.md"
    - "docs/scenarios/NON-TRADING-WINDOW-RUNNER-READINESS.md"
    - "docs/scenarios/MULTIFACTOR-SIMULATION-RUNNER-OPERATION.md"
    - "docs/USER-MANUAL.md"
    - "docs/components/RUNNER.md"
  must_read:
    - "process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-VERIFICATION-2026-06-25.md"
    - "process/STATE.md"
    - "process/state/STATE.current.json"
  read_if_needed:
    - "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-READONLY-EVIDENCE-2026-06-25.json"
    - "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-RUNTIME-EVIDENCE-2026-06-25.json"
    - "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-RUNTIME-EVIDENCE-INDEX-2026-06-25.json"
    - "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-PHASE-GOALS-2026-06-25.md"
    - "process/runbooks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-RUNBOOK-2026-06-25.md"
  do_not_read_by_default:
    - "process/changes/CR-*.md"
    - "process/stories/*-LLD.md"
    - "process/docs/design/*"
    - "process/docs/quality/*"
    - "/tmp/*"
    - ".env"
    - ".env.*"
    - "/home/hyde/.quant-lab/runtime/**"
---

# Runner QMT Simulation Multifactor Next Session Handoff

## 1. 下一轮目标

用户将清除会话后继续推进。下一轮目标是：

```text
完成所有非交易窗口期可以完成的任务；交易窗口期才能完成的任务只记录清楚，不执行。
```

下一轮默认不需要再验证 `session_expired`，因为 after-restart 复验已通过。除非用户明确授权 runtime 操作，否则下一轮不得启动 gateway、读取 env、连接真实账户、执行 submit/cancel 或触达 `small_live/live`。

## 2. 当前最新事实

| 项目 | 最新状态 |
|---|---|
| gateway session | 已恢复，`session_ready=true` |
| signed positions readonly | 已通过，`position_count_bucket=one_to_ten` |
| real multifactor runtime operator | 已用临时最小 spec 跑通 P1-P4 |
| P3 simulation submit/cancel | `submitted_count=1`，`cancelled_count=1`，`unknown_count=0` |
| P4 reconciliation | `pass`，`unresolved_count=0` |
| evidence redaction | pass |
| small_live/live | 未授权，未触达 |
| 正式 StrategyAdmissionPackage | 未固定 |
| 正式 operator spec | 未固定 |
| 稳定性窗口 | 仅完成临时 spec 单次验证；长期窗口未完成 |

必须读取的最新验证记录：

```text
process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-VERIFICATION-2026-06-25.md
```

## 3. 重要限制

本次 runtime pass 使用临时最小 spec：

```text
/tmp/runner-qmt-multifactor-operator-after-restart-riskpass-20260625.json
```

旧的原始临时 spec 已不存在：

```text
/tmp/runner-qmt-multifactor-operator-spec-20260625.json
```

因此下一轮不能声称“正式策略包已通过日常运行验收”。正确表述是：

```text
runtime 链路已通过临时 spec 证明可通；正式 StrategyAdmissionPackage、正式 operator spec、稳定性窗口仍待完成。
```

## 4. 非交易窗口任务清单

下一轮应按下列顺序执行，除非用户另有更新指令。

| 顺序 | 任务 | 产出 | 交易窗口需求 |
|---:|---|---|---|
| 1 | 更新状态文件，清除旧 `session_expired` 阻断 | `process/STATE.md`、`process/state/STATE.current.json` 更新 | 否 |
| 2 | 固化 after-restart evidence 摘要 | 确认 `process/evidence/*AFTER-RESTART*` 已存在并通过 redaction 检查 | 否 |
| 3 | 生成或选定正式 StrategyAdmissionPackage | 正式 package 路径和摘要 | 否 |
| 4 | 生成正式 operator spec 模板 | 不含凭据的 JSON 模板，字段完整 | 否 |
| 5 | 跑正式输入 `preflight-only` | evidence/index，`runtime_touched=false` | 否 |
| 6 | 跑正式输入 `plan-only` | P1 target + P2 order plan evidence | 否 |
| 7 | 跑正式输入 `fixture` | P1/P2/P4 fixture evidence，P3 no-op | 否 |
| 8 | 跑正式输入 `reconcile-only` | 对账 schema evidence | 否 |
| 9 | 定义稳定性窗口标准 | N 次 / N 日、pass/fail、异常退出标准 | 否 |
| 10 | 补 manual takeover checklist | unknown / cancel unknown / recon diff / session expired 处理清单 | 否 |
| 11 | 补 kill-switch 判定表 | 停止新单、撤单、人工接管条件 | 否 |
| 12 | 补证据保留策略 | 保存范围、脱敏粒度、保留周期、Git 排除边界 | 否 |
| 13 | 设计 operator command / runbook-driven entry | 手动日常命令或调度入口草案 | 否 |
| 14 | 刷新用户手册和 runbook | 文档反映最新状态和下一步 | 否 |
| 15 | 跑回归测试 | runner/QMT/docs guardrail 通过 | 否 |

## 5. 交易窗口任务清单

这些任务必须等交易窗口或明确 runtime 授权后执行；下一轮仅记录，不执行。

| 顺序 | 任务 | 通过标准 | 备注 |
|---:|---|---|---|
| 1 | 用正式 StrategyAdmissionPackage 跑真实 runtime operator | P1-P4 pass | 需要 simulation runtime authorization |
| 2 | 用真实 current positions 生成 P2 order plan | 订单计划基于真实只读持仓摘要 | evidence 只保留脱敏摘要 |
| 3 | 验证正式 buy 路径 | submit/cancel accepted，unknown=0 | 仅 simulation |
| 4 | 验证正式 sell 路径 | submit/cancel accepted，unknown=0 | 需要可卖数量条件 |
| 5 | 验证 post positions 对账 | recon pass，unresolved=0 | 差异进入 manual takeover |
| 6 | 验证订单回执状态 | accepted/rejected/unknown 处理明确 | unknown 必须阻断下一轮 |
| 7 | 验证日终撤未成 | 未完成订单撤单或人工记录 | 需要真实订单状态条件 |
| 8 | 累计稳定性窗口 | 连续 N 次 / N 个交易日通过 | N 值由非交易窗口任务冻结 |
| 9 | 运行异常演练 | session_expired、gateway down、risk blocked、cancel unknown、recon diff 可 fail closed | 不得绕过 gate |
| 10 | gateway 停止与恢复验收 | stop confirmed，下一轮启动正常 | 操作手册闭环 |

## 6. 禁止事项

下一轮如果没有新的用户明确授权，禁止执行：

- 启动或重启 QMT gateway。
- 读取 runtime env 内容或凭据。
- 执行真实 signed positions readonly。
- 执行 runtime operator。
- 执行 simulation submit/cancel。
- 进入 `small_live`、`live`、live-readonly、scale-up。
- 保存账号原文、证券代码原文、原始 broker order ref、资金明细、raw broker payload。
- provider fetch、lake write、broker lake write、NAS、publish、remote Git 写入。

## 7. 推荐恢复步骤

清会话后的第一条动作建议：

```text
读取 process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-NEXT-SESSION-HANDOFF-2026-06-25.md，
然后完成第 4 节中所有非交易窗口任务。
```

如果状态文件仍显示 `session_expired` 阻断，以本 handoff 和 after-restart verification 为准，先更新状态文件再继续。
