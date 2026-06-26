---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-VERIFICATION-2026-06-25"
task_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-2026-06-25"
created_at: "2026-06-25T16:22:35+08:00"
owner: "host-orchestrator"
stage_decision: "PASS_WITH_SCOPE_LIMIT"
runtime_authorization_ref: "runner-qmt-simulation-smoke-20260625-sim"
readonly_evidence_ref: "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-READONLY-EVIDENCE-2026-06-25.json"
runtime_evidence_ref: "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-RUNTIME-EVIDENCE-2026-06-25.json"
runtime_evidence_index_ref: "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-RUNTIME-EVIDENCE-INDEX-2026-06-25.json"
---

# Runner QMT Simulation Multifactor After-Restart Verification

## 1. 结论

| 项目 | 结果 |
|---|---|
| 阶段决策 | `PASS_WITH_SCOPE_LIMIT` |
| gateway health | `PASS`，`session_ready=true`，`runtime_status=xtquant-ready` |
| signed positions readonly | `PASS`，`position_count_bucket=one_to_ten`，`items_redacted_count=1` |
| session_expired 阻断 | 已解除 |
| runtime operator | `PASS`，使用临时最小 simulation spec |
| P1 target portfolio | `generated`，`selected_count=1` |
| P2 order plan | `generated`，`order_count=1`，risk pass |
| P3 simulation submit/cancel | `submitted_count=1`，`cancelled_count=1`，`unknown_count=0` |
| P4 reconciliation | `pass`，`unresolved_count=0`，`manual_takeover_required=false` |
| stability window | 本次临时 spec 单次 `pass`；长期 N 次 / N 日未完成 |
| small_live / live | `NOT_AUTHORIZED`，未触达 |
| evidence redaction | `PASS`，未保存 raw account、raw symbol、raw broker ref、raw payload、secret/token、fund detail |

本记录用于替代旧的 `session_expired` 阻断判断：用户重启 Windows QMT gateway 后，signed positions readonly 与真实 multifactor simulation runtime operator 均已通过。

## 2. 范围限制

本次通过的 runtime operator 使用临时最小 spec：

```text
/tmp/runner-qmt-multifactor-operator-after-restart-riskpass-20260625.json
```

旧的原始临时 spec 不存在：

```text
/tmp/runner-qmt-multifactor-operator-spec-20260625.json
```

因此，本次结论证明 runtime 链路和 runner P1-P4 可通，但不等价于“正式 StrategyAdmissionPackage + 正式当前持仓输入 + 日常参数”的稳定运行验收。

## 3. 复验过程摘要

| 步骤 | 结果 | 证据 |
|---|---|---|
| gateway health | PASS | health 返回 `session_ready=true` |
| capabilities | PASS | endpoint matrix 包含 positions、simulation submit、simulation cancel |
| CR138 signed positions readonly | PASS | `readonly_evidence_ref` |
| runtime operator 第一次复跑 | BLOCKED | `pretrade_risk_blocked`，原因是临时 spec 使用 unsupported `source_kind` |
| runtime operator 第二次复跑 | PASS | `runtime_evidence_ref` |

第一次 runtime operator 已越过 pre-positions session gate，但因本地风控阻断：

```text
blocked_reason=pretrade_risk_blocked
order blocked_reason=unsupported_snapshot_source
```

将临时 spec 的 `risk_snapshot.source_kind` 改为 `sanitized_snapshot` 后，P1-P4 完整通过。

## 4. Evidence 摘要

| Evidence | 路径 | 说明 |
|---|---|---|
| readonly evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-READONLY-EVIDENCE-2026-06-25.json` | health / capabilities / positions readonly 脱敏摘要 |
| runtime evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-RUNTIME-EVIDENCE-2026-06-25.json` | P1-P4 runtime operator 脱敏摘要 |
| runtime evidence index | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-RUNTIME-EVIDENCE-INDEX-2026-06-25.json` | evidence schema validation |

关键计数：

| 字段 | 值 |
|---|---:|
| `submitted_count` | 1 |
| `cancelled_count` | 1 |
| `rejected_count` | 0 |
| `unknown_count` | 0 |
| `unresolved_count` | 0 |
| `manual_takeover_required` | false |
| `small_live_or_live_authorized` | false |

## 5. 后续处理

下一轮清会话后，主目标不是继续修 gateway，而是完成所有非交易窗口任务：

1. 把本次 after-restart 结论纳入当前状态和下一轮 handoff。
2. 生成或选定正式 StrategyAdmissionPackage。
3. 生成正式 operator spec 模板。
4. 用正式输入跑 `preflight-only`、`plan-only`、`fixture`、`reconcile-only`。
5. 定义并记录长期稳定性窗口标准。
6. 刷新 runbook、manual takeover、kill-switch 和证据保留策略。

交易窗口任务仍需保留：

1. 用正式 StrategyAdmissionPackage 和真实 current positions 跑一次真实 runtime operator。
2. 验证正式 buy / sell 路径、post positions、订单回执和对账。
3. 累计 N 次 / N 个交易日稳定性窗口。
4. 完成日终撤未成、gateway 停止 / 恢复和异常演练。

## 6. 安全边界

- 本记录不授权 `small_live` / `live`。
- 本记录不授权 provider fetch、lake write、broker lake write、NAS、publish 或 remote Git 写入。
- 本记录不保存 env 内容、凭据、HMAC secret、账号原文、证券代码原文、原始 broker order ref、资金明细或原始 broker payload。
