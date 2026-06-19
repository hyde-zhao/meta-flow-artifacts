---
check_id: "CP7-CR099-RUNTIME-AUTHORIZED-SMOKE-RERUN-PASS"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-19T16:54:06+08:00"
target:
  cr_id: "CR-099"
  run_id: "cr099-runner-readonly-smoke-20260619-sim-001"
  authorization_ref: "cr099-runner-readonly-smoke-20260619-sim"
  base_url: "http://172.30.32.1:18765"
  evidence_path: "/home/hyde/.quant-lab/evidence/qmt/cr099/redacted/cr099-runner-readonly-smoke-20260619-sim-001/evidence.json"
  previous_attempt: "process/checks/CP7-CR099-RUNTIME-AUTHORIZED-SMOKE-ATTEMPT-2026-06-19.md"
---

# CP7 CR099 Runtime Authorized Smoke Rerun Pass

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| process route health | PASS | `uv run --python 3.11 meta-flow workspace check` | `process_link_health: ok` |
| Windows gateway 已重启 | PASS | 用户回复“我已经重新启动起来了” | 用户手动完成 Windows 端 gateway / session 刷新 |
| 单次授权仍在同一 run 范围 | PASS | 当前对话 + CR099 runtime contract | 复跑同一 `authorization_ref` / `run_id`，只读 endpoint allowlist 不变 |
| 上次阻断可重试 | PASS | `CP7-CR099-RUNTIME-AUTHORIZED-SMOKE-ATTEMPT-2026-06-19.md` | 上次仅因 `session_expired` 阻断，forbidden counters 全 0 |

## Checklist

| # | 检查项 | 状态 | 证据 | 说明 |
|---|---|---|---|---|
| 1 | health 只读检查 | PASS | redacted evidence | `status=ok`，`session_ready=true`，`raw_payload_emitted=false` |
| 2 | capabilities 只读检查 | PASS | redacted evidence | `status=ok`，`readonly_supported=true`，`raw_payload_emitted=false` |
| 3 | query_positions_readonly | PASS | redacted evidence | `status=ok`，`position_count_bucket=zero`，`items_redacted_count=0`，`raw_payload_emitted=false` |
| 4 | evidence schema | PASS | CR099 checker | checker `passed=true`，errors=0，warnings=0 |
| 5 | forbidden counters | PASS | redacted evidence | `account_id_values`、`security_code_values`、`raw_quantity_values`、`raw_cash_values`、`order_fields`、`fill_fields`、`submit_cancel_calls`、`buy_sell_calls`、`nas_operations`、`provider_lake_publish` 全部为 0 |
| 6 | 权限最小化 | PASS | 命令范围 | 只连接 `http://172.30.32.1:18765`，只调用 health / capabilities / query_positions_readonly |
| 7 | 原始信息泄露 | PASS | evidence / checker | 未输出 Windows `.env`、HMAC secret、账户原文、证券代码、数量、资金、委托、成交或原始日志 |

## 执行命令摘要

| 命令 | 结果 | 说明 |
|---|---|---|
| `uv run --python 3.11 meta-flow workspace check` | PASS | process route 正常 |
| `uv run --python 3.11 python scripts/collect_cr099_runner_readonly_smoke.py --env-file /home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template --base-url http://172.30.32.1:18765 --authorization-ref cr099-runner-readonly-smoke-20260619-sim --run-id cr099-runner-readonly-smoke-20260619-sim-001 ...` | PASS | 采集器 exit 0；只输出 checker 摘要 |
| `uv run --python 3.11 python scripts/check_cr099_redacted_evidence.py --evidence ... --json` | PASS | evidence schema `passed=true` |

## Redacted Evidence Summary

| 字段 | 值 |
|---|---|
| evidence path | `/home/hyde/.quant-lab/evidence/qmt/cr099/redacted/cr099-runner-readonly-smoke-20260619-sim-001/evidence.json` |
| overall_status | `pass` |
| health | `status=ok`，`session_ready=true` |
| capabilities | `status=ok`，`readonly_supported=true` |
| query_positions_readonly | `status=ok`，`position_count_bucket=zero`，`items_redacted_count=0` |
| positions_digest | `sha256:b728d0998dc83fecce65f9b7a1d4b34f580d643743546b1cc05f02506aaf5d6c` |
| forbidden counters | 全部 `0` |
| raw_payload_emitted | 全部 `false` |

## Exit Criteria

| 条目 | 状态 | 说明 |
|---|---|---|
| runner -> Windows gateway 真实只读 smoke 成功 | PASS | 三个 endpoint 均通过，positions 返回脱敏空持仓摘要 |
| 安全边界保持 | PASS | forbidden counters 全 0；无 raw payload 输出 |
| 可进入 CP8 | PASS | CR099 runtime 风险已关闭到当前授权范围；非空 / 交易日路径仍为独立 follow-up 风险 |

## 结论

- 结论：`PASS`
- CR099 目标范围内的真实 runner readonly smoke 已通过。
- 剩余风险：当前为模拟账户 / 空持仓路径；非空持仓或交易日复测不在本 run 范围内，需另行授权。
- 不授权项仍保持：NAS、Windows `.env`、账户原文、交易写、simulation/live、provider/lake/publish。
