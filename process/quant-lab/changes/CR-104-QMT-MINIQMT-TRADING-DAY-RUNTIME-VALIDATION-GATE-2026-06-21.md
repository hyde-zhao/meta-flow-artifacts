---
cr_id: "CR-104"
cr_kind: "runtime-authorization"
lifecycle_status: "closed"
readiness_status: "ready_with_risk"
gate_status: "closed"
gate_profile: "runtime"
status: "closed-current-delivery"
impact_level: "high"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
fast_lane_upgrade_reason: "命中真实 QMT/MiniQMT/XtQuant/gateway runtime、账户只读、交易日 query_positions、simulation/order-write 设计边界，必须使用 standard/runtime gate。"
rollback_to: "CR103 non-trading-day validation PASS_WITH_RISK; CR101 closed-current-delivery / READY_WITH_RISK"
approval_result: "cp8-approved-user-20260622-close-readonly-runtime-pass"
created_at: "2026-06-21T13:31:01+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-22T11:24:03+08:00"
source: "user"
linked_issue: ""
parent_cr: "CR-101"
source_checkpoint: "process/checkpoints/CP8-CR101-DELIVERY-READINESS.md"
source_decision_id: "DQ-CP8-CR101-04"
follow_up_type: "runtime-authorization-trading-day"
risk_class: "qmt-miniqmt-trading-day-runtime"
owner: "host-orchestrator"
revisit_condition: "2026-06-22 交易日且用户明确批准 CP2 runtime scope、执行主机、证据脱敏、停止条件和不授权项。"
acceptance_criteria: "CP2 仅批准明日交易日验证范围后，才允许进入后续 CP3/CP5；未获 CP2 明确批准前，不启动 QMT/MiniQMT/XtQuant/gateway，不读取 env/凭据/账户，不查询账户/持仓/委托/成交，不 submit/cancel，不 simulation/live。"
close_condition: "MiniQMT/gateway readonly runtime 验证已覆盖 zero 与 one_to_ten 持仓 bucket，CP8 由用户要求收尾并推送。"
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "active"
reframed_by: []
reframe_summary: ""
---

# CR-104 QMT/MiniQMT Trading-Day Runtime Validation Gate

## 变更描述

用户要求将必须等交易日才能验证的 QMT/MiniQMT 内容另起 CR，并计划在 2026-06-22 由用户配合完成验证。

本 CR 当前只创建 runtime gate 和输入清单。**未获 CP2 明确批准前，不执行任何真实 runtime 或账户动作。**

## 目标范围

| 范围 | 目标 | 当前状态 |
|---|---|---|
| QMT direct-run validation | 证明 QMT terminal 能加载当前 `qmt_terminal_direct` 策略包 target 或最小入口 | deferred / not covered by this closure；如仍需要，另起独立 gate |
| MiniQMT / gateway readonly validation | 证明 health / capabilities / query_positions_readonly 的受控只读链路 | PASS |
| redacted evidence | 保存 run_id、status、capabilities summary、positions digest / bucket、forbidden counters，不保存原始账户/持仓/日志 | PASS |
| order-write / simulation / live | 当前只列为后续设计项，不在 CP2 默认授权 | not authorized；需 CR105 |

## 明日需要用户准备的信息

| 字段 | 建议值 / 说明 | 是否可贴到对话 |
|---|---|---|
| `run_id` | `cr104-trading-day-runtime-20260622-<HHMM>` | 可以 |
| `execution_host_label` | `windows-qmt-host` / 你自己的脱敏机器标签 | 可以 |
| `account_mode` | `simulated` / `paper` / `readonly-real`，推荐先用模拟或只读 | 可以 |
| `qmt_terminal_status` | 已打开 / 未打开 / 由你手工打开 | 可以 |
| `gateway_mode` | no-gateway / existing-gateway / user-manual-windows-side | 可以 |
| `authorization_ref` | `user-approved-cr104-<run_id>` | 可以 |
| `credential_source` | `user-held-not-shared` 或 `none-needed` | 可以；不要贴凭据 |
| `evidence_output_mode` | `redacted-summary-only` | 可以 |
| `allowed_endpoints` | `health`, `capabilities`, `query_positions_readonly` | 可以 |
| `abort_conditions` | raw payload、credential leak、order endpoint、submit/cancel、unexpected account field、runtime error | 可以 |

## 当前默认不授权项

- 不读取 `.env`、token、password、HMAC secret、cookie、session、private key、QMT 账号或交易密码。
- 不粘贴账户号、持仓明细、委托、成交、资金、证券代码、日志原文或截图原文。
- 不执行 submit、cancel、buy、sell。
- 不执行 simulation/live/order-write。
- 不执行 provider fetch、lake write、catalog publish。
- 不访问 NAS，除非另起独立 NAS gate。

## CP2 待决策项草案

| 决策 ID | 类型 | 问题 | 推荐方案 | 备选 | 风险 / 回退 |
|---|---|---|---|---|---|
| DQ-CP2-CR104-01 | runtime_authorization | 是否授权 2026-06-22 交易日只读 runtime 验证？ | 只授权 health / capabilities / query_positions_readonly，redacted-summary-only。 | 只做 QMT direct-run load；延期；取消。 | 若出现 raw payload 或凭据需求，立即停止。 |
| DQ-CP2-CR104-02 | security | 凭据如何处理？ | 用户线下持有，不贴对话；agent 不读取 env/凭据。 | 用户手工执行并只回传摘要。 | 凭据泄露风险；默认 fail closed。 |
| DQ-CP2-CR104-03 | scope | 是否包含 order-write / simulation / live？ | 不包含，只保留后续 FU-CR101-001。 | 另起高风险设计门禁。 | 避免从只读验证跳到交易写。 |
| DQ-CP2-CR104-04 | evidence | evidence 保存什么？ | run_id、endpoint status、count bucket、digest、forbidden counters。 | pass/fail only；截图人工审查。 | 不保存 raw account/log。 |

## 交易日执行前置条件

| 条件 | 当前状态 |
|---|---|
| CP2 runtime scope approved | pending |
| execution host ready | user required |
| credential handled offline | user required |
| redacted evidence template confirmed | pending |
| abort conditions accepted | pending |
| QMT/MiniQMT/gateway runtime authorization | not authorized yet |

## Prepared Execution Scripts

| Script | Purpose | Runtime boundary |
|---|---|---|
| `scripts/run_cr104_qmt_miniqmt_validation.ps1` | Windows-side operator entrypoint. Default `DryRunPreflight` creates and checks blocked-preflight evidence only. | Default mode does not start QMT/MiniQMT/XtQuant/gateway, does not read `.env` / credentials / account, does not query positions, and does not submit/cancel/sim/live. |
| `scripts/build_cr104_runtime_preflight_evidence.py` | Builds CR099-compatible redacted blocked-preflight evidence for today's trial run. | Imports only standard library; no runtime, network, credential, account or trading modules. |
| `scripts/check_cr099_redacted_evidence.py` | Existing redacted evidence checker reused by CR104 scripts. | Static file validation only. |

Today's trial command is documented in `docs/features/cr104-qmt-miniqmt-trading-day-validation/INPUT-CHECKLIST.md`. `ReadonlyRuntime` mode remains blocked until CR104 CP2 explicitly approves runtime authorization.

## Runtime Readonly Validation Result

| Field | Value |
|---|---|
| run_id | `cr104-miniqmt-readonly-20260621-cr099-reuse-001` |
| authorization_ref | `cr099-runner-readonly-smoke-20260619-sim` |
| base_url | `http://172.30.32.1:18765` |
| evidence_path | `/home/hyde/.quant-lab/evidence/qmt/cr104/redacted/cr104-miniqmt-readonly-20260621-cr099-reuse-001/evidence.json` |
| checker_status | `passed=true`, `errors=[]`, `warnings=[]` |
| health | `status=ok`, `session_ready=true` |
| capabilities | `status=ok`, `readonly_supported=true` |
| query_positions_readonly | `status=ok`, `position_count_bucket=zero`, `items_redacted_count=0`, `raw_payload_emitted=false` |
| forbidden counters | all `0` |
| result | `PASS_WITH_RISK` |

本次验证由用户明确要求复用 CR099 会话参数和 `--runtime-authorization-ref cr099-runner-readonly-smoke-20260619-sim` 发起；实际执行只调用 health / capabilities / query_positions_readonly。未启动 QMT/MiniQMT/gateway，未执行 submit/cancel/buy/sell/simulation/live，未访问 NAS，未 provider/lake/catalog publish。剩余风险：本次返回 `position_count_bucket=zero`，证明只读链路与脱敏 schema 成功，但不证明非空持仓脱敏路径。

## Runtime Readonly Retest Result

| Field | Value |
|---|---|
| checked_at | `2026-06-22` |
| run_id | `cr104-miniqmt-readonly-20260622-cr099-reuse-002` |
| authorization_ref | `cr099-runner-readonly-smoke-20260619-sim` |
| evidence_path | `/home/hyde/.quant-lab/evidence/qmt/cr104/redacted/cr104-miniqmt-readonly-20260622-cr099-reuse-002/evidence.json` |
| checker_status | `passed=true`, `errors=[]`, `warnings=[]` |
| health | `status=ok`, `session_ready=true` |
| capabilities | `status=ok`, `readonly_supported=true` |
| query_positions_readonly | `status=ok`, `position_count_bucket=zero`, `items_redacted_count=0`, `raw_payload_emitted=false` |
| forbidden counters | all `0`; `submit_cancel_calls=0`, `buy_sell_calls=0` |
| result | `PASS_WITH_RISK` |

本次复测确认 CR099 会话复用下的 MiniQMT/gateway 只读链路仍可用，但持仓仍为空。用户提出的 `ETF300` 约 10000 元买入验证属于 order-write / simulation 范围，不属于 CR104；现有 runtime server 也未暴露 submit/cancel 路由，相关动作必须转入后续 CR105 order-write gate。

## Runtime Nonzero Position Readonly Retest Result

| Field | Value |
|---|---|
| checked_at | `2026-06-22` |
| run_id | `cr104-miniqmt-readonly-nonzero-20260622-cr099-reuse-003` |
| authorization_ref | `cr099-runner-readonly-smoke-20260619-sim` |
| evidence_path | `/home/hyde/.quant-lab/evidence/qmt/cr104/redacted/cr104-miniqmt-readonly-nonzero-20260622-cr099-reuse-003/evidence.json` |
| checker_status | `passed=true`, `errors=[]`, `warnings=[]` |
| health | `status=ok`, `session_ready=true` |
| capabilities | `status=ok`, `readonly_supported=true` |
| query_positions_readonly | `status=ok`, `position_count_bucket=one_to_ten`, `items_redacted_count=1`, `positions_digest_present=true`, `raw_payload_emitted=false` |
| forbidden counters | all `0`; `account_id_values=0`, `security_code_values=0`, `raw_quantity_values=0`, `raw_cash_values=0`, `order_fields=0`, `fill_fields=0`, `submit_cancel_calls=0`, `buy_sell_calls=0` |
| result | `PASS` |

本次复测在用户确认已有持仓后执行，证明 CR104 只读链路已覆盖非空持仓脱敏路径：只输出 bucket、digest 和 redacted item count，不输出账户、证券代码、数量、资金、委托、成交或日志原文。CR104 只读范围可进入 CP8 收口；order-write / simulation 仍需后续 CR105 独立门禁和实现。

## 下一步

CR104 当前状态为 `closed-current-delivery / READY_WITH_RISK`。本次关闭接受 readonly runtime PASS，并明确 QMT direct-run 未覆盖、order-write / simulation/live 未授权。当前仍不授权 submit/cancel/buy/sell/simulation/live、账户原文、日志原文、NAS 或 provider/lake/catalog publish。若需要测试模拟账户下单 / 撤单，应启动 CR105 order-write 独立门禁。
