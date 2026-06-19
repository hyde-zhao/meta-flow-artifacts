---
check_id: "CR097-RUNTIME-PREFLIGHT"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-19T10:55:00+08:00"
checked_at: "2026-06-19T11:42:47+08:00"
target:
  cr_id: "CR-097"
  runtime_mode: "windows_gateway"
  windows_gateway_host: "172.30.32.1"
  windows_gateway_port: 18765
  wsl_client_ip: "172.30.33.29"
  scope:
    - health
    - capabilities
    - query_positions_auth_negative
    - query_positions_authenticated
---

# CR097 Runtime Preflight 检查结果

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CR097 active | PASS | `process/changes/CR-097-REAL-QMT-READONLY-RUNTIME-SMOKE-PER-RUN-AUTHORIZATION-2026-06-19.md` | 已启动 runtime preflight |
| 用户确认 Windows gateway | PASS | 当前对话 | 用户声明 Windows 端启动成功，并说明当前为模拟新账户、预期空持仓、今日非交易日 |
| WSL 网络参数 | PASS | `ip route get 1.1.1.1` / `ip addr show eth0` | Windows gateway `172.30.32.1`，WSL client `172.30.33.29` |
| XtQuant import / session 修复 | PASS | `/qmt/health` | health 返回 `status=ok`、`session_ready=true`、`runtime_status=xtquant-ready` |
| HMAC client env 授权 | PASS | 当前对话结构化授权 | 用户授权读取 `/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template` 并执行一次 CR097 `query_positions` 只读成功路径 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | WSL 到 Windows gateway 端口连通 | PASS | `timeout 5 nc -vz 172.30.32.1 18765` | 连接成功 |
| 2 | `/qmt/health` 可读 | PASS | `curl --max-time 10 -s http://172.30.32.1:18765/qmt/health` | health 返回 `status=ok`、`session_ready=true`、`session_state=ready`、`runtime_status=xtquant-ready`，响应为脱敏配置摘要 |
| 3 | `/qmt/capabilities` 可读 | PASS | `curl --max-time 10 -s http://172.30.32.1:18765/qmt/capabilities` | capabilities contract 返回；`operation_authorized=false`、`real_operation=false`、`simulation/live/order/cancel=false` |
| 4 | positions 未签名负向请求 | PASS | `POST /qmt/account/positions` without HMAC | 返回 `blocked_reason=auth_header_missing`；`adapter_call=0`、`readonly_positions_adapter_call=0`、`query_positions_read_attempt=0`、`raw_positions_emit=0`，证明 auth gate 前 fail-closed |
| 5 | 只读 query_positions 成功路径 | PASS | `uv run --with typer --python 3.11 python -m trading.qmt_runtime_cli query-positions --env-file /home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template --base-url http://172.30.32.1:18765 --run-id cr097-readonly-smoke-20260619-sim-001 --request-id cr097-readonly-smoke-20260619-sim-001-query-positions --timeout-seconds 10` | 返回 `status=ok`、`readonly_query_authorized=true`、`position_count=0`、`items_redacted=[]`、`raw_payload_emitted=false`、`operation_authorized=false`、`real_operation=false` |
| 6 | 凭据 / 账户 / NAS 边界 | PASS | 本检查 | 仅读取用户授权的最小 HMAC client env；未读取 Windows `.env`、未输出账号 / secret / 持仓原文、未访问 NAS，未执行交易写 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 可进入 query_positions 成功路径 | PASS | health `session_ready=true`; negative auth gate PASS; authenticated query PASS | runtime ready 且成功路径只读查询通过 |
| fail-closed 边界 | PASS | unsigned positions request | 未签名请求不会触发 adapter 或账户查询 |
| 脱敏 evidence 完成 | PASS | `/home/hyde/.quant-lab/evidence/qmt/cr097/redacted/cr097-query-positions-redacted-20260619.json` | 只保存空持仓脱敏摘要和安全计数，不保存原始 CLI 输出 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Runtime preflight result | `process/checks/CR097-RUNTIME-PREFLIGHT-2026-06-19.md` | PASS | 已记录 |
| Redacted runtime preflight evidence | `/home/hyde/.quant-lab/evidence/qmt/cr097/redacted/cr097-runtime-preflight-redacted-20260619.json` | PASS | 仅保存 health/capabilities/负向 auth gate 摘要；不保存原始持仓或 secret |
| WSL client env template | `/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template` | READY | 仅列最小 C 端 HMAC 字段；不包含 QMT 账号、MiniQMT 或 XtQuant 路径 |
| Redacted query_positions evidence | `/home/hyde/.quant-lab/evidence/qmt/cr097/redacted/cr097-query-positions-redacted-20260619.json` | PASS | 成功路径脱敏 evidence；空持仓、无原始明细 |

## 结论

- 结论：`PASS`
- 已通过项：Windows gateway 可达；QMT runtime ready；health / capabilities 可读且脱敏；未签名 positions 请求在 auth gate 前 fail-closed；授权后的 `query_positions_readonly` 成功路径返回空持仓脱敏结果。
- 成功路径结果：`position_count=0`、`items_redacted=[]`、`raw_payload_emitted=false`、`redaction_status=pass`、`operation_authorized=false`、`real_operation=false`。
- 下一步：进入 CR097 收敛，生成 CP6 / CP7 / CP8 或等价 release readiness 过程证据；继续禁止读取 Windows `.env`、账号原文、持仓原文、NAS、交易写、provider/lake/publish。
