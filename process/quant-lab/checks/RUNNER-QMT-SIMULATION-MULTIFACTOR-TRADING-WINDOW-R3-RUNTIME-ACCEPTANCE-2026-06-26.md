---
check_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R3-RUNTIME-ACCEPTANCE-2026-06-26"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-26T10:45:42+08:00"
owner: "host-orchestrator"
status: "PASS_WITH_STABILITY_WINDOW_COLLECTING"
runtime_authorization_granted: true
runtime_authorization_ref: "runner-qmt-simulation-multifactor-trading-window-20260626-r3-auth"
run_id: "runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3"
runtime_touched: true
small_live_or_live_authorized: false
---

# Runner QMT Simulation Multifactor Trading Window R3 Runtime Acceptance

## 1. 结论

用户已授权本次逐次 `simulation runtime authorization`，并已启动 Windows gateway。r3 使用私有 runtime overlay 生成的正式 runtime spec / admission package 执行完成，结论为 `PASS_WITH_STABILITY_WINDOW_COLLECTING`。

本轮已经覆盖并通过：

- P0 gateway health / runtime identity：`status=ok`、`session_ready=true`、`runtime_status=xtquant-ready`、`runtime_mode=simulation`、`account_kind=simulation`、`runtime_profile=cr138-simulation`。
- P0.5 signed positions readonly：`status=ok`，positions 条数为 0，未发现原始证券代码、账号、token、secret 或 raw payload 泄漏。
- P1 target：通过。
- P2 order plan：通过，`order_count=1`。
- P3 simulation execution：通过，`submitted_count=1`、`cancelled_count=1`、`rejected_count=0`、`unknown_count=0`、`manual_takeover_required=false`。
- P4 reconciliation：通过，`unresolved_count=0`、`drift_bucket=drift:pass`、`manual_takeover_required=false`。

本轮未授权、未执行 `small_live` / `live`，未触达 NAS、provider/lake/catalog、publish 或 remote Git。

## 2. 验收矩阵

| 阶段 | 结果 | 说明 |
|---|---|---|
| P0 gateway health | `PASS` | gateway 在 Windows 运行，WSL 通过 `http://172.30.32.1:18765` 访问；identity 为 simulation / CR138 |
| P0 capabilities | `PASS_WITH_CONTRACT_VIEW` | endpoint matrix 包含 positions、simulation submit、simulation cancel；capabilities unsigned view 中的 authorization 字段不作为 P0 阻断 |
| P0.5 signed positions readonly | `PASS` | readonly evidence 写入成功，positions 条数为 0 |
| P1 target | `PASS` | 私有 overlay 替换 fixture instrument 后可生成 target |
| P2 order plan | `PASS` | `order_count=1` |
| P3 execution | `PASS` | simulation submit/cancel 各 1 次；无 rejected / unknown；无需人工接管 |
| P4 reconciliation | `PASS` | `unresolved_count=0`，对账漂移桶为 pass |
| Stability window | `COLLECTING` | `completed_runs=2`、`required_runs=5`，仍需后续逐次授权累计 |
| Redaction | `PASS` | index 声明未保存 raw account、raw symbol、raw broker order ref、raw payload、secret/token、fund detail；扫描未命中原始 ETF 代码或敏感字段 |

## 3. 证据

| Evidence | 路径 |
|---|---|
| r3 readonly evidence | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-READONLY-EVIDENCE-2026-06-26-r3.json` |
| r3 runtime operator evidence | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3/operator-evidence.json` |
| r3 runtime evidence index | `process/evidence/runner-simulation-runtime-2026-06-26/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3/operator-evidence.index.json` |
| r3 private runtime overlay | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3-runtime-overlay.json` |
| r3 private runtime spec | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3-runtime-spec.json` |
| r3 private runtime admission | `/home/hyde/.quant-lab/runtime/qmt/cr138-simulation/runner-qmt-simulation-multifactor-trading-window-2026-06-26-r3-runtime-admission-package.json` |

## 4. 脱敏与边界

- 私有 overlay 中的真实可提交证券代码未写入正式 evidence。
- runtime evidence 中的 instrument / broker order / cancel / reconciliation 值均为 `symbol:*`、`broker-order:*`、`cancel-ref:*`、`orders:*` 等 hash/ref 形态。
- evidence index 的 `redaction_summary` 为：
  - `raw_account_saved=false`
  - `raw_symbol_saved=false`
  - `raw_broker_order_ref_saved=false`
  - `raw_payload_saved=false`
  - `secret_or_token_saved=false`
  - `fund_detail_saved=false`
- 本轮扫描未命中私有 overlay 的原始 ETF 代码、`INSTRUMENT_*`、账号字段、secret/password 或 `raw_payload=true`。

## 5. 剩余风险与后续动作

| 风险 / 动作 | 状态 | 说明 |
|---|---|---|
| r1/r2 fixture instrument runtime 阻断 | `CLOSED` | r3 私有 runtime 输入已通过 P3/P4，旧 `transport_timeout` / `validation_blocked` 已关闭 |
| gateway 侧 fixture symbol 防护 | `ACTIVE_GUARDRAIL` | 仍保留，防止 `INSTRUMENT_*` 或 fixture ref 误入 submit/cancel |
| 长期稳定性窗口 | `COLLECTING` | 当前 `completed_runs=2`、`required_runs=5`；后续每次都必须重新取得逐次 simulation runtime authorization |
| Windows gateway | `USER_MANAGED` | 本轮 runtime 已完成；如无需继续验证，用户可以停止 gateway |

下一步建议：不要在同一授权下继续重复 submit/cancel。若需要累计稳定性窗口，请在下一次交易窗口重新授权 simulation runtime，并从 P0 health / identity 重新开始。
