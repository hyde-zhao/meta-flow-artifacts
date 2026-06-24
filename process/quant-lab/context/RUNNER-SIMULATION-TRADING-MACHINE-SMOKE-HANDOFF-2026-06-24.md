---
handoff_id: "RUNNER-SIMULATION-TRADING-MACHINE-SMOKE-HANDOFF-2026-06-24"
created_at: "2026-06-24T18:03:29+08:00"
from_agent: "host-orchestrator"
to_agent: "meta-qa"
target_phase: "story-execution"
target_status: "simulation-trading-machine-smoke-ready"
source_work_package: "RUNNER-MODEL-SIMULATION-LIVE-ENTRY-2026-06-24"
semantic: "stage-dispatch"
status: "handoff-created"
dispatch:
  required: false
  semantic: "stage-dispatch"
  mode: "handoff-only"
  platform: "codex"
  agent_role: "meta-qa"
  canonical_role: "meta-qa"
  codex_agent_name: "meta-qa"
  reasoning_profile: "default"
  dispatch_trigger: "simulation-trading-machine-smoke"
  agent_path: ""
  tool_name: ""
  agent_id: ""
  agent_name: ""
  thread_id: ""
  spawned_at: ""
  resumed_at: ""
  completed_at: ""
  evidence: ""
  fallback_reason: "current request is handoff and push only; no subagent dispatch requested"
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

当前状态：`simulation-entry-ready`

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

交接状态：`ready-for-simulation-trading-machine-smoke`

下一阶段推荐目标包名：

`RUNNER-QMT-SIMULATION-TRADING-MACHINE-SMOKE-2026-06-24`
