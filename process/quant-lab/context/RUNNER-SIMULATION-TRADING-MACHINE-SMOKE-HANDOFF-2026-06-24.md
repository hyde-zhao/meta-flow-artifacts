---
handoff_id: "RUNNER-SIMULATION-TRADING-MACHINE-SMOKE-HANDOFF-2026-06-24"
created_at: "2026-06-24T18:03:29+08:00"
from_agent: "host-orchestrator"
to_agent: "meta-qa"
target_phase: "story-execution"
target_status: "simulation-smoke-passed"
source_work_package: "RUNNER-MODEL-SIMULATION-LIVE-ENTRY-2026-06-24"
semantic: "stage-dispatch"
status: "completed"
result_status: "PASS"
verification_ref: "process/checks/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-VERIFICATION-2026-06-24.md"
evidence_ref: "process/evidence/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-EVIDENCE-2026-06-24.json"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-qa"
  canonical_role: "meta-qa"
  codex_agent_name: "meta-qa-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "risk-review"
  agent_path: ""
  tool_name: "spawn_agent"
  agent_id: "019efc4a-3592-7431-aa6e-028ffca4b75f"
  agent_name: "qa-critical-lv"
  thread_id: "019efc4a-3592-7431-aa6e-028ffca4b75f"
  spawned_at: "2026-06-25T08:59:50+08:00"
  resumed_at: ""
  completed_at: "2026-06-25T09:04:52+08:00"
  evidence: "spawn_agent"
  fallback_reason: ""
  approved_by: ""
  approved_at: ""
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  structured_choice_allowed: false
  allowed_question_scope: "simulation smoke run parameters that do not expose credentials"
  forbidden_question_scope: "credentials, secrets, unrestricted live trading authorization, real-money submit/cancel approval"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/RUNNER-MODEL-SIMULATION-LIVE-ENTRY-WORK-PACKAGE-2026-06-24.md"
  context_ref: "process/context/RUNNER-MODEL-SIMULATION-LIVE-ENTRY-WORK-PACKAGE-2026-06-24.md"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 12
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
  allowed_reads:
    - "process/context/RUNNER-MODEL-SIMULATION-LIVE-ENTRY-WORK-PACKAGE-2026-06-24.md"
    - "process/checks/RUNNER-MODEL-SIMULATION-LIVE-ENTRY-VERIFICATION-2026-06-24.md"
    - "trading/strategy_runner/simulation_activation.py"
    - "trading/qmt_runtime.py"
    - "trading/qmt_client.py"
    - "trading/qmt_gateway_contracts.py"
    - "tests/test_runner_simulation_activation.py"
    - "tests/test_cr020_runtime_manual_validation.py"
    - "tests/test_cr020_linux_client_rest_transport.py"
  must_read:
    - "process/context/RUNNER-MODEL-SIMULATION-LIVE-ENTRY-WORK-PACKAGE-2026-06-24.md"
    - "process/checks/RUNNER-MODEL-SIMULATION-LIVE-ENTRY-VERIFICATION-2026-06-24.md"
  read_if_needed:
    - "trading/strategy_runner/simulation_activation.py"
    - "trading/qmt_runtime.py"
    - "trading/qmt_client.py"
    - "trading/qmt_gateway_contracts.py"
    - "tests/test_runner_simulation_activation.py"
    - "tests/test_cr020_runtime_manual_validation.py"
    - "tests/test_cr020_linux_client_rest_transport.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/changes/CR-*.md"
    - "process/stories/*"
    - "process/archive/*"
    - "full conversation transcript"
    - "credentials, tokens, secrets, account raw files"
---

# Runner Simulation Trading Machine Smoke Handoff

## 1. 当前结论

当前目标包按新的 meta-flow 流程推进，不沿用旧 CR 拆分队列。源码层面已完成 runner 从模型盘目标组合进入 QMT 模拟账户真实操作入口的实现切片，并通过本地自动化验证。

当前状态：`simulation-smoke-passed`

下一目标状态：`simulation-smoke-passed`

## 2. 已完成范围

- `trading/strategy_runner/simulation_activation.py` 新增模型盘目标组合到 simulation order 的受控激活入口。
- `trading/qmt_gateway_contracts.py` 新增 simulation submit/cancel endpoint、scope、请求对象和脱敏 operation payload。
- `trading/qmt_runtime.py` 新增 QMT runtime simulation submit/cancel HTTP route、HMAC scope 和 XtQuant `order_stock` / `cancel_order_stock` adapter 调用。
- `trading/qmt_client.py` 新增 `allow_simulation_transport`，默认 fail-closed，显式开启后 simulation submit/cancel 才走 REST transport。
- 结果脱敏已覆盖账号、原始证券代码、原始 broker order ref 和 secret。
- process hygiene 已识别本轮新目标包资产，`unclassified=0`。

## 3. 已验证范围

最后一次本地验证结果：

- runner / QMT / stage gate / pretrade risk / OMS / endpoint / hygiene 相关回归：`109 passed`
- `scripts/check_process_artifact_hygiene.py --json`：PASS，`unclassified=0`
- `py_compile`：PASS
- `git diff --check`：PASS

验证记录见：

- `process/checks/RUNNER-MODEL-SIMULATION-LIVE-ENTRY-VERIFICATION-2026-06-24.md`

## 4. 下一阶段任务

下一阶段只做交易机模拟账户 smoke，不授权真实资金 live submit/cancel。

建议执行顺序：

1. 在交易机准备 MiniQMT / XtQuant 环境变量。
2. 启动 QMT gateway。
3. 验证 gateway health 和 QMT session ready。
4. 执行 positions readonly。
5. 构造最小目标组合，执行 `activate_simulation_orders()`。
6. 使用 `QmtClientConfig(allow_simulation_transport=True)` 发起 simulation submit。
7. 对返回的脱敏 broker order ref 发起 simulation cancel。
8. 检查 response / log / evidence 不泄露账号、证券代码、原始订单号或 secret。
9. 写入 simulation smoke 脱敏 evidence 和验证结论。

## 5. 阻断条件

任一条件命中时必须停止，不得进入实盘：

- QMT session 未 ready。
- HMAC scope 不匹配或授权缺失。
- stage gate 缺 runbook、kill switch、reconciliation 或 per-run authorization。
- pretrade risk 失败。
- response / log 出现账号、证券代码、原始订单号或 secret。
- simulation submit/cancel 没有可解释的 QMT adapter 结果。
- 用户未给出独立真实资金授权却尝试进入 `small_live` / `live`。

## 6. 不授权范围

本交接不授权：

- 真实资金 `small_live` / `live` submit。
- 真实资金 cancel。
- 放宽 `allow_simulation_transport` 默认值。
- 复用 simulation scope 作为 live scope。
- 保存或推送任何凭据、token、账号原文、订单原文或资金明细原文。

## 7. 进入实盘前必须新增的工作

模拟盘 smoke 通过后，若要进入小额实盘，必须另起高风险授权门：

- 独立 `small_live` / `live` endpoint 和 HMAC scope。
- 独立 client 显式开关，默认 false。
- 单笔、单日、标的、资金上限。
- kill switch / cancel-only / 回滚方案。
- live readonly 验证。
- 小额实盘人工授权和验证证据。

## 8. 交接判定

交接状态：`completed`

阶段结论：`PASS`

子 agent 已完成允许边界内的本地准备验证。2026-06-25 09:32:35 +08:00 用户授权后，Host Orchestrator 对已启动的交易机 gateway 做了 health / capabilities 探测；2026-06-25 09:45:26 +08:00 用户重启 gateway 后复测，两次均因 runtime 配置未加载阻塞。2026-06-25 09:53:54 +08:00 用户修正 env 后，WSL 侧确认 gateway health 为 `session_ready=true`、`runtime_status=xtquant-ready`。2026-06-25 10:05:34 +08:00，WSL 侧 HMAC client env 可用，signed positions readonly 通过；signed simulation submit endpoint 使用故意无效 payload 验证 auth/session/endpoint 到达，并在 validation 阶段被拦截，未触达下单 adapter。

2026-06-25 10:31:33 +08:00 首次真实 simulation submit 已 accepted 并返回 `cancel-ref`，但 cancel 被本地 validation 阻断。随后完成 cancel-ref 映射和数字型 XtQuant order id 撤单兼容回修，已同步到 `C:/quant-lab-runtime` 并由用户重启 gateway。2026-06-25 10:38:00 +08:00 signed buy simulation submit/cancel smoke 通过：buy submit accepted，cancel accepted，cancel adapter 返回 `xtquant-simulation-cancel-accepted`。2026-06-25 10:43:00 +08:00 按用户追加授权补测 signed sell simulation submit/cancel smoke，通过：sell submit accepted，cancel accepted，cancel adapter 返回 `xtquant-simulation-cancel-accepted`。

- `process/checks/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-VERIFICATION-2026-06-24.md`
- `process/evidence/RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-EVIDENCE-2026-06-24.json`

真实交易机 simulation smoke 已覆盖 buy/sell 双向 submit/cancel 并通过，状态推进为 `simulation-smoke-passed`。本交接仍不授权真实资金 `small_live` / `live` submit/cancel，不授权保存或推送真实凭据、账号原文、证券代码原文、原始订单引用或资金明细。

下一阶段推荐目标包名：

`RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-2026-06-24`
