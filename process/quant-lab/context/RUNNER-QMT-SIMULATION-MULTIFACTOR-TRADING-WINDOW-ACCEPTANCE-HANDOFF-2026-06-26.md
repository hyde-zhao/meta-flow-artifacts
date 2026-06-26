---
handoff_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-ACCEPTANCE-HANDOFF-2026-06-26"
created_at: "2026-06-26T00:00:00+08:00"
from: "host-orchestrator"
to: "next-host-orchestrator"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
semantic: "context-reset-before-trading-window-acceptance"
status: "ready-for-trading-window-authorization"
runtime_authorization_granted: false
runtime_touched: false
small_live_or_live_authorized: false
dispatch:
  required: false
  semantic: "stage-dispatch"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "next-host-orchestrator"
  canonical_role: "host-orchestrator"
  codex_agent_name: ""
  reasoning_profile: "default"
  dispatch_trigger: "context-reset-before-trading-window-acceptance"
  agent_path: ""
  tool_name: ""
  agent_id: ""
  agent_name: ""
  thread_id: ""
  spawned_at: ""
  resumed_at: ""
  completed_at: ""
  evidence: "handoff-only; no runtime action and no subagent dispatch executed"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: true
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "Ask only for exact per-run simulation runtime authorization, non-sensitive runtime refs, host/base-url placeholders, run_id, and confirmation that Windows QMT gateway session is ready."
  forbidden_question_scope: "credentials, raw account data, raw positions, raw symbols, raw broker order refs, funds detail, token/secret/session values, NAS, provider/lake/catalog publish, small_live/live authorization"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-ACCEPTANCE-HANDOFF-2026-06-26.md"
  context_ref: "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-ACCEPTANCE-HANDOFF-2026-06-26.md"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 14
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: "Trading-window acceptance requires precise authorization boundary, runtime execution order and failure split after session reset."
  allowed_reads:
    - "process/state/STATE.current.json"
    - "process/STATE.md"
    - "process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-NON-TRADING-WINDOW-COMPLETION-2026-06-25.md"
    - "process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-VERIFICATION-2026-06-25.md"
    - "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-STRATEGY-ADMISSION-PACKAGE-2026-06-25.json"
    - "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-OPERATOR-SPEC-2026-06-25.json"
    - "docs/reference/RUNNER-QMT-AUTHORIZATION.md"
    - "docs/QMT-GATEWAY-INSTALL.md"
    - "docs/scenarios/MULTIFACTOR-SIMULATION-RUNNER-OPERATION.md"
    - "docs/scenarios/NON-TRADING-WINDOW-RUNNER-READINESS.md"
    - "docs/USER-MANUAL.md"
    - "docs/components/RUNNER.md"
    - "process/runbooks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-RUNBOOK-2026-06-25.md"
  must_read:
    - "process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-ACCEPTANCE-HANDOFF-2026-06-26.md"
    - "process/state/STATE.current.json"
    - "process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-NON-TRADING-WINDOW-COMPLETION-2026-06-25.md"
    - "docs/reference/RUNNER-QMT-AUTHORIZATION.md"
    - "docs/QMT-GATEWAY-INSTALL.md"
    - "docs/scenarios/MULTIFACTOR-SIMULATION-RUNNER-OPERATION.md"
  read_if_needed:
    - "process/evidence/runner-simulation-formal-2026-06-25/preflight-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json"
    - "process/evidence/runner-simulation-formal-2026-06-25/plan-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json"
    - "process/evidence/runner-simulation-formal-2026-06-25/fixture/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json"
    - "process/evidence/runner-simulation-formal-2026-06-25/reconcile-only/runner-qmt-simulation-multifactor-formal-non-trading-2026-06-25/operator-evidence.index.json"
    - "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-READONLY-EVIDENCE-2026-06-25.json"
    - "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-RUNTIME-EVIDENCE-2026-06-25.json"
    - "process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-AFTER-RESTART-RUNTIME-EVIDENCE-INDEX-2026-06-25.json"
  do_not_read_by_default:
    - ".env"
    - ".env.*"
    - "/home/hyde/.quant-lab/runtime/**"
    - "/tmp/*"
    - "process/changes/CR-*.md"
    - "process/stories/*-LLD.md"
    - "process/docs/design/*"
    - "process/docs/quality/*"
    - "data/**"
    - "reports/**"
---

# Runner QMT Simulation Multifactor Trading Window Acceptance Handoff

## 1. 下一轮目标

用户将清除上下文后开启交易窗口验收。下一轮目标是：

```text
在用户给出新的逐次 simulation runtime authorization 后，用正式 StrategyAdmissionPackage 和授权窗口内的真实 current positions 脱敏摘要执行一次真实 QMT simulation multifactor runtime operator 验收；未授权前只读取本 handoff、准备授权问题和检查清单，不执行任何 runtime 动作。
```

本 handoff 不构成授权，不允许直接启动 gateway、读取 env、查询账户、提交 / 撤单或触达 `small_live` / `live`。

## 2. 当前基线事实

| 项目 | 最新状态 |
|---|---|
| 非交易窗口任务 | 已完成，`PASS_WITH_TRADING_WINDOW_TASKS_DEFERRED` |
| 正式 StrategyAdmissionPackage | 已冻结：`process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-STRATEGY-ADMISSION-PACKAGE-2026-06-25.json` |
| 正式 operator spec | 已冻结：`process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-OPERATOR-SPEC-2026-06-25.json` |
| 离线 `preflight-only` | `pass`，`runtime_touched=false` |
| 离线 `plan-only` | `pass`，`runtime_touched=false` |
| 离线 `fixture` | `pass`，`submitted_count=0`，`cancelled_count=0` |
| 离线 `reconcile-only` | `pass`，`unresolved_count=0` |
| after-restart runtime 链路 | 2026-06-25 已用临时最小 spec 复验通过 |
| 稳定性窗口 | 标准已冻结：`5` 次真实 simulation run，覆盖 `3` 个交易日；当前真实累计仍待开始 |
| small_live / live | 未授权，保持 deferred |

关键结论：

```text
runtime 链路已通过临时 spec 证明可通；正式 StrategyAdmissionPackage、正式 operator spec 已通过非交易窗口离线验证；交易窗口验收仍必须重新取得逐次授权并使用授权窗口内真实 current positions 脱敏摘要。
```

## 3. 下一轮启动时必须先问用户的授权问题

下一轮第一步不是执行命令，而是让用户给出明确授权。建议原样询问：

```text
请确认是否授权本次交易窗口 simulation runtime 验收。

请用明确文字确认以下范围：
1. 仅授权本次 RUNNER-QMT-SIMULATION-MULTIFACTOR simulation 验收；
2. 授权 scope 包含 gateway_start、port_bind、account_readonly、simulation、submit_cancel、gateway_stop；
3. 授权读取本机私有 runtime env，但不得展示或保存 env 内容；
4. 授权查询 simulation account positions，但 evidence 只保存脱敏摘要；
5. 授权执行正式 StrategyAdmissionPackage 的 P1-P4 runtime operator；
6. 授权 simulation submit/cancel 仅限本次 runner 生成的 order intents；
7. 不授权 small_live、live、scale_up、provider fetch、lake write、broker lake write、NAS、publish、remote Git 写入；
8. 若出现 session_expired、unknown order、cancel unknown、recon diff 或 redaction fail，立即 fail closed 并进入 manual takeover。

同时请提供本次 runtime_authorization_ref、授权窗口起止时间、Windows gateway host/base-url、WSL client env 路径引用、Windows server env 路径引用和操作者确认。
不要提供任何密码、token、secret、账号原文、资金明细、证券代码原文或 broker 原始回执。
```

只有用户明确确认后，才能进入第 5 节执行。

## 4. 授权前禁止事项

未获得新的逐次 runtime authorization 前，禁止执行：

- 启动或重启 QMT gateway。
- 读取 `.env` / runtime env 内容。
- 查询真实或 simulation account positions。
- 执行 `--mode runtime`。
- 执行 simulation submit/cancel。
- 执行 signed positions readonly。
- 触达 `small_live`、`live`、live-readonly、scale-up。
- provider fetch、lake write、broker lake write、NAS、publish、remote Git 写入。
- 保存账号原文、证券代码原文、原始 broker order ref、资金明细、raw broker payload、token、secret、session。

## 5. 授权后执行顺序

### P0：Gateway 和授权边界检查

1. 读取授权确认文本，记录 `runtime_authorization_ref`、窗口时间、scope、forbidden commands。
2. Windows S 端 diagnostics：确认 `runtime_mode=simulation`、`account_kind=simulation`、`runtime_profile=cr138-simulation` 或本次授权 profile。
3. Windows S 端启动 gateway。
4. C 端检查 health。
5. C 端检查 capabilities，必须包含 positions、simulation submit、simulation cancel。
6. 若任一步失败，停止，不进入 P1-P4。

### P0.5：Signed Positions Readonly

1. 使用授权窗口内 WSL client env 和 base-url 运行 positions readonly。
2. evidence 只保存 digest、bucket、items_redacted、instrument_ref、redaction_status。
3. 禁止保存 raw positions、账号、证券代码原文、精确持仓、资金明细、session、broker raw payload。
4. 若 `session_expired` 或 redaction fail，停止并进入失败分流。

### P1-P4：正式 Runtime Operator

1. 基于正式 StrategyAdmissionPackage 和授权窗口内真实 current positions 脱敏摘要生成 runtime operator 输入。
2. 执行 `scripts/run_qmt_multifactor_simulation_operator.py --mode runtime`。
3. P1 必须生成 target portfolio。
4. P2 必须生成 order plan，或明确 risk blocked 且不进入 P3。
5. P3 必须只提交 / 撤销本次 runner 生成的 simulation order intents。
6. P4 必须完成 post positions reconciliation。
7. 写入 runtime evidence 和 index。

### P5：停止和收尾

1. 正常退出 operator。
2. 停止 Windows gateway。
3. 记录 gateway stop confirmed。
4. 检查无 unknown order、无未关闭 manual takeover、无未关闭 recon diff。
5. 写本次 trading-window acceptance check。

## 6. 推荐命令模板

以下命令只在授权后使用；其中 env 路径、base-url 和 authorization ref 由用户提供，不得在文档中写入真实 secret 内容。

### Windows S 端 diagnostics

```bash
uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli server-diagnostics \
  --env-file <windows-runtime-env> \
  --runtime-authorization-ref <runtime-authorization-ref>
```

### Windows S 端启动 gateway

```bash
uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli serve \
  --env-file <windows-runtime-env> \
  --runtime-authorization-ref <runtime-authorization-ref>
```

### C 端 positions readonly

```bash
uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli query-positions \
  --env-file <wsl-client-env> \
  --base-url <gateway-base-url> \
  --run-id <run-id> \
  --request-id <request-id>
```

### Runtime operator

```bash
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/run_qmt_multifactor_simulation_operator.py \
  --mode runtime \
  --spec-json <runtime-operator-spec-json> \
  --strategy-admission-json process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-STRATEGY-ADMISSION-PACKAGE-2026-06-25.json \
  --env-file <wsl-client-env> \
  --base-url <gateway-base-url> \
  --output-dir process/evidence/runner-simulation-runtime-<date>
```

注意：`process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-OPERATOR-SPEC-2026-06-25.json` 是非交易窗口模板，`authorization_ref=""`，且 current positions 是 fixture。交易窗口 runtime 前必须生成本次私有 runtime operator spec，填入授权 ref 和授权窗口内真实 current positions 脱敏输入；不得把 raw positions 写入仓库。

## 7. 通过标准

| 阶段 | 通过标准 |
|---|---|
| P0 gateway | diagnostics / health / capabilities 均 pass，profile 为 simulation |
| P0.5 readonly | positions readonly pass，redaction pass，raw payload 未保存 |
| P1 target | target portfolio generated，`selected_count >= 1` |
| P2 order plan | order plan generated，risk pass；若 risk blocked，则不得进入 P3 |
| P3 execution | submit/cancel accepted 或明确 rejected；`unknown_count=0` |
| P4 reconciliation | `reconciliation_status=pass`，`unresolved_count=0`，`manual_takeover_required=false` |
| stop | gateway stop confirmed |
| evidence | raw account / raw symbol / raw broker ref / secret / fund detail 均未保存 |

验收通过后，只能计入稳定性窗口 `1/5`；不能直接声明 daily runtime acceptance。

## 8. 失败分流

| 失败 | 立即动作 | 后续处理 |
|---|---|---|
| authorization missing / expired | 不执行 runtime | 要求用户重新授权 |
| diagnostics fail | 不启动或停止 gateway | 检查 Windows S 端环境 |
| `session_expired` | 停止 runner，不进入 P3 | 重新登录 / 重启 gateway 后从 health/capabilities/readonly 开始 |
| capabilities missing | 不进入 P3 | 修 endpoint matrix / scope |
| readonly redaction fail | 丢弃 evidence | 修脱敏后重跑 readonly |
| risk blocked | 不 submit/cancel | 调整策略输入或风险参数后重新 P1/P2 |
| submit unknown | 停止新单 | MiniQMT 人工核对，manual takeover |
| cancel unknown | 停止新 submit，避免重复未知撤单 | 人工核对 broker 状态 |
| recon diff | 停止下一轮自动运行 | kill-switch candidate；差异关闭后才能恢复 |
| raw evidence leak | 丢弃 evidence，不发布 | 修 evidence writer / redaction |

## 9. 验收后应写入的产物

建议命名：

| 产物 | 建议路径 |
|---|---|
| trading-window acceptance check | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-ACCEPTANCE-2026-06-26.md` |
| runtime readonly evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26.json` |
| runtime operator evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-RUNTIME-EVIDENCE-2026-06-26.json` |
| runtime evidence index | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-RUNTIME-EVIDENCE-INDEX-2026-06-26.json` |
| gateway stop evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-GATEWAY-STOP-EVIDENCE-2026-06-26.json` |
| next stability window state | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-2026-06-26.md` |

所有 evidence 必须是脱敏摘要，不得保存 raw runtime payload。

## 10. 验收后状态更新

若本次 trading-window acceptance 通过：

- `STATE.current.json.next_action.type` 改为 `runner-qmt-simulation-multifactor-stability-window-collecting`。
- 稳定性窗口计数：`completed_runs=1`，目标 `required_runs=5`、`required_trading_days=3`。
- 保持 `small_live_or_live_authorized=false`。

若失败：

- `STATE.current.json.blocked` 是否置 true 取决于失败类型。
- `unknown`、`recon_diff`、redaction fail 应进入 manual takeover / incident。
- 不得继续下一轮 runtime，直到失败关闭。

## 11. 下一轮建议第一句话

清上下文后，建议用户第一句输入：

```text
读取 process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-ACCEPTANCE-HANDOFF-2026-06-26.md，按文档准备交易窗口验收。先不要执行 runtime，先向我确认逐次 simulation runtime authorization。
```
