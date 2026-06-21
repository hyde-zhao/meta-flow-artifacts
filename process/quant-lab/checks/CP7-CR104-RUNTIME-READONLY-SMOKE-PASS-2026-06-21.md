---
check_id: "CP7-CR104-RUNTIME-READONLY-SMOKE-PASS"
status: "PASS_WITH_RISK"
owner: "host-orchestrator"
created_at: "2026-06-21T14:05:00+08:00"
target:
  cr_id: "CR-104"
  run_id: "cr104-miniqmt-readonly-20260621-cr099-reuse-001"
  authorization_ref: "cr099-runner-readonly-smoke-20260619-sim"
  base_url: "http://172.30.32.1:18765"
  evidence_path: "/home/hyde/.quant-lab/evidence/qmt/cr104/redacted/cr104-miniqmt-readonly-20260621-cr099-reuse-001/evidence.json"
---

# CP7 CR104 Runtime Readonly Smoke Pass

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 用户授权 | PASS | 用户指令 | 用户要求复用 CR099 内容，并指定 `--runtime-authorization-ref cr099-runner-readonly-smoke-20260619-sim` 尝试运行需要验证的接口。 |
| 运行范围 | PASS | 命令参数 | 仅调用 `health` / `capabilities` / `query_positions_readonly`。 |
| 禁止项 | PASS | 命令范围与 checker | 不执行 submit/cancel/buy/sell/simulation/live，不访问 NAS，不 provider/lake/catalog publish。 |
| evidence 输出 | PASS | `scripts/check_cr099_redacted_evidence.py` | 只保存 redacted evidence 摘要。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 说明 |
|---|---|---|---|---|
| 1 | health 只读检查 | PASS | redacted evidence | `status=ok`，`session_ready=true`。 |
| 2 | capabilities 只读检查 | PASS | redacted evidence | `status=ok`，`readonly_supported=true`。 |
| 3 | query_positions_readonly | PASS_WITH_RISK | redacted evidence | `status=ok`，`position_count_bucket=zero`，`items_redacted_count=0`，`raw_payload_emitted=false`。 |
| 4 | evidence schema | PASS | checker output | checker `passed=true`，`errors=[]`，`warnings=[]`。 |
| 5 | forbidden counters | PASS | redacted evidence summary | `account_id_values`、`security_code_values`、`raw_quantity_values`、`raw_cash_values`、`order_fields`、`fill_fields`、`submit_cancel_calls`、`buy_sell_calls`、`nas_operations`、`provider_lake_publish` 全部为 `0`。 |
| 6 | 原始信息泄露 | PASS | checker output | 未输出账号、资金、证券代码、数量、市值、委托、成交、凭据或原始日志。 |

## Executed Commands

| 命令 | 结果 | 说明 |
|---|---|---|
| `uv run --python 3.11 python scripts/collect_cr099_runner_readonly_smoke.py --env-file /home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template --base-url http://172.30.32.1:18765 --authorization-ref cr099-runner-readonly-smoke-20260619-sim --run-id cr104-miniqmt-readonly-20260621-cr099-reuse-001 --timeout-seconds 10 --output-json /home/hyde/.quant-lab/evidence/qmt/cr104/redacted/cr104-miniqmt-readonly-20260621-cr099-reuse-001/evidence.json` | PASS | 采集器 exit 0；只输出 checker 摘要。 |
| `uv run --python 3.11 python scripts/check_cr099_redacted_evidence.py --evidence <evidence> --json` | PASS | `passed=true`，`errors=[]`，`warnings=[]`。 |

## Redacted Evidence Summary

| 字段 | 值 |
|---|---|
| overall_status | `pass` |
| execution_mode | `codex-runner-authorized` |
| health_status | `ok` |
| health_session_ready | `true` |
| capabilities_status | `ok` |
| capabilities_readonly_supported | `true` |
| query_positions_status | `ok` |
| position_count_bucket | `zero` |
| items_redacted_count | `0` |
| raw_payload_emitted | `false` |
| forbidden_counters | all `0` |

## Exit Criteria

| 条目 | 状态 | 说明 |
|---|---|---|
| MiniQMT/gateway readonly interface smoke | PASS_WITH_RISK | 三个只读 endpoint 均通过；positions 为 zero bucket。 |
| 脱敏 evidence 可审计 | PASS | checker 通过且无 warning。 |
| 安全边界保持 | PASS | forbidden counters 全 0；无 raw payload。 |
| 可进入 CP8 | PASS | 可准备 CR104 CP8 收口确认。 |

## 结论

结论：`PASS_WITH_RISK`。

CR104 当前范围内的真实 MiniQMT/gateway 只读接口验证已通过。剩余风险为本次 `query_positions_readonly` 返回 zero-position bucket，不覆盖非空持仓脱敏路径；也不授权任何交易写、simulation/live、账户原文、日志原文、NAS 或 provider/lake/catalog publish。
