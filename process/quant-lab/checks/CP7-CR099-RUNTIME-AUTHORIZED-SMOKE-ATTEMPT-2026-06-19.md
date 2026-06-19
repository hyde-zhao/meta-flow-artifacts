---
check_id: "CP7-CR099-RUNTIME-AUTHORIZED-SMOKE-ATTEMPT"
status: "BLOCKED_RUNTIME_SESSION_EXPIRED"
owner: "host-orchestrator"
created_at: "2026-06-19T16:38:43+08:00"
target:
  cr_id: "CR-099"
  run_id: "cr099-runner-readonly-smoke-20260619-sim-001"
  authorization_ref: "cr099-runner-readonly-smoke-20260619-sim"
  base_url: "http://172.30.32.1:18765"
  evidence_path: "/home/hyde/.quant-lab/evidence/qmt/cr099/redacted/cr099-runner-readonly-smoke-20260619-sim-001/evidence.json"
---

# CP7 CR099 Runtime Authorized Smoke Attempt

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| process route health | PASS | `uv run --python 3.11 meta-flow workspace check` | `process_link_health: ok` |
| 用户运行授权 | PASS | 当前对话 | 用户说明 Windows 已启动、服务器地址和脱敏文件参考 CR097、client env 使用 CR097 验证内容、evidence 输出目录按建议 |
| CR097 参数恢复 | PASS | `process/checks/CR097-RUNTIME-PREFLIGHT-2026-06-19.md` | `base_url=http://172.30.32.1:18765`，client env path=`/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template` |
| CR099 采集器存在 | PASS | `scripts/collect_cr099_runner_readonly_smoke.py` | WSL client-side 采集器；不改 Windows gateway，不启动服务 |
| 采集器本地测试 | PASS | `tests/test_cr099_runner_real_readonly_smoke_contract.py` | `9 passed` |

## Checklist

| # | 检查项 | 状态 | 证据 | 说明 |
|---|---|---|---|---|
| 1 | health 只读检查 | PASS | redacted evidence | `status=ok`，`session_ready=true`，`raw_payload_emitted=false` |
| 2 | capabilities 只读检查 | PASS | redacted evidence | `status=ok`，`readonly_supported=true`，`raw_payload_emitted=false` |
| 3 | query_positions_readonly | BLOCKED | redacted evidence | `status=blocked`，`reason_code=session_expired` |
| 4 | forbidden counters | PASS | redacted evidence | CR099 要求的 10 个 forbidden counters 全部为 0 |
| 5 | evidence schema | PASS | `scripts/check_cr099_redacted_evidence.py --evidence ...` | checker `passed=true`，无 errors / warnings |
| 6 | 原始 payload / secret 输出 | PASS | evidence / 命令输出审查 | 未输出 HMAC secret、Windows `.env`、账户原文、证券代码、数量、资金、委托、成交或原始日志 |
| 7 | 服务端变更需要 | N/A | 本次结果 | 未发现需要修改 Windows gateway 代码；当前阻断为服务端 session 过期状态 |

## 执行命令摘要

| 命令 | 结果 | 说明 |
|---|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr099_runner_real_readonly_smoke_contract.py` | PASS | `9 passed` |
| `uv run --python 3.11 python scripts/collect_cr099_runner_readonly_smoke.py --env-file /home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template --base-url http://172.30.32.1:18765 --authorization-ref cr099-runner-readonly-smoke-20260619-sim --run-id cr099-runner-readonly-smoke-20260619-sim-001 ...` | exit 2 | evidence schema PASS；业务状态 `overall_status=blocked` |
| `uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli query-positions --env-file /home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template --base-url http://172.30.32.1:18765 ...` | exit 0 / blocked response | 直接复核服务端返回 `reason_code=session_expired`；forbidden counters 仍为 0 |

## Redacted Evidence Summary

| 字段 | 值 |
|---|---|
| evidence path | `/home/hyde/.quant-lab/evidence/qmt/cr099/redacted/cr099-runner-readonly-smoke-20260619-sim-001/evidence.json` |
| overall_status | `blocked` |
| health | `status=ok`，`session_ready=true` |
| capabilities | `status=ok`，`readonly_supported=true` |
| query_positions_readonly | `status=blocked`，`reason_code=session_expired` |
| forbidden counters | 全部 `0` |
| raw_payload_emitted | 全部 `false` |

## Exit Criteria

| 条目 | 状态 | 说明 |
|---|---|---|
| runner -> Windows gateway 真实只读 smoke 成功 | FAIL | positions 被 `session_expired` 阻断 |
| 安全边界保持 | PASS | 未越过只读范围；forbidden counters 全 0 |
| 可继续复跑 | PASS_WITH_ACTION | 需要用户在 Windows 端刷新 / 重启 gateway session 后复跑同一 CR099 命令 |

## 结论

- 结论：`BLOCKED_RUNTIME_SESSION_EXPIRED`
- 当前不需要 GitHub server-side push；Windows gateway 已可响应 health / capabilities，阻断点是服务端 positions session 过期。
- 用户侧最小动作：在 Windows 端刷新 QMT session 或重启 gateway runtime，使 `/qmt/account/positions` 不再返回 `session_expired`；完成后通知 host-orchestrator 复跑同一 CR099 run。
- 不授权项仍保持：NAS、Windows `.env`、账户原文、交易写、simulation/live、provider/lake/publish。
