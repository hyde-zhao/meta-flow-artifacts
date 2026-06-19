---
checkpoint_id: "CP7"
checkpoint_name: "CR097 Runtime Smoke Verification Done"
type: "rolling_auto"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-19T11:42:47+08:00"
checked_at: "2026-06-19T11:42:47+08:00"
target:
  phase: "story-execution"
  story_id: "CR097-REAL-QMT-READONLY-RUNTIME-SMOKE"
  artifacts:
    - "process/checks/CP6-CR097-REAL-QMT-READONLY-RUNTIME-SMOKE-CODING-DONE.md"
    - "process/checks/CR097-RUNTIME-PREFLIGHT-2026-06-19.md"
    - "/home/hyde/.quant-lab/evidence/qmt/cr097/redacted/cr097-query-positions-redacted-20260619.json"
---

# CP7 CR097 Runtime Smoke Verification Done 检查结果

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP6 PASS | PASS | `process/checks/CP6-CR097-REAL-QMT-READONLY-RUNTIME-SMOKE-CODING-DONE.md` | runtime smoke evidence 已准备 |
| 验证模式明确 | PASS | CR097 | `runtime`，但 scope 限定为 health、capabilities、query_positions_readonly |
| 用户 runtime 授权 | PASS | 当前对话 | 用户授权真实 QMT runtime、最小 HMAC client env 和一次 query_positions 成功路径 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | 验证对象清单完整 | PASS | CR097 / CP6 | 覆盖 Windows gateway、health、capabilities、auth negative gate、authenticated query_positions、redacted evidence |
| 2 | 追踪矩阵完整 | PASS | CR092-FU-02 -> CR097 -> CP6 evidence -> CP7 result | runtime smoke follow-up 已转正式 CR 并形成证据 |
| 3 | health 契约通过 | PASS | `/qmt/health` 摘要 | `status=ok`、`session_ready=true`、`runtime_status=xtquant-ready` |
| 4 | capabilities 边界通过 | PASS | `/qmt/capabilities` 摘要 | `operation_authorized=false`、`real_operation=false`、交易写仍未授权 |
| 5 | auth fail-closed 通过 | PASS | 未签名 positions 请求 | `auth_header_missing`，adapter / query counters 为 0 |
| 6 | query_positions 成功路径通过 | PASS | redacted query evidence | `position_count=0`、`items_redacted=[]`、`raw_payload_emitted=false`、`redaction_status=pass` |
| 7 | 安全边界验证 | PASS | evidence / counters | forbidden operation counters 为 0；仅 `query_positions_read_attempt=1` 和 `readonly_positions_adapter_call=1` 属于授权只读成功路径 |
| 8 | 剩余风险分级 | PASS | 本文件 | 模拟新账户、空持仓、非交易日不覆盖非空持仓或交易日行为，作为后续观察风险记录 |

## 自动化 / Runtime 验证证据

| 命令 / 请求 | 结果 | 摘要 |
|---|---|---|
| `timeout 5 nc -vz 172.30.32.1 18765` | PASS | TCP connect succeeded |
| `GET /qmt/health` | PASS | `status=ok`; `session_ready=true`; `runtime_status=xtquant-ready` |
| `GET /qmt/capabilities` | PASS | endpoint contract 可读；operation/live/simulation/order/cancel 未授权 |
| unsigned `POST /qmt/account/positions` | PASS | `blocked_reason=auth_header_missing`; adapter/query counters 为 0 |
| `qmt_runtime_cli query-positions` with authorized minimal env | PASS | `status=ok`; `position_count=0`; `items_redacted=[]`; `raw_payload_emitted=false` |

## Agent Dispatch Evidence

| 检查项 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 子 agent 调度模式 | WAIVED | 当前对话 | runtime smoke 由 host-orchestrator inline 执行，以保持用户逐步授权和人工中止边界 |
| canonical role | WAIVED | `host-orchestrator` | 未启动 meta-qa |
| Codex custom agent | WAIVED | N/A | 不适用 |
| reasoning profile | WAIVED | N/A | 不适用 |
| dispatch trigger | WAIVED | CR097 runtime authorization | 高风险外部 runtime 验证 |
| agent 标识 | WAIVED | N/A | 不适用 |
| 平台工具证据 | WAIVED | inline-fallback | 无 subagent 工具调用 |
| 完成时间 | PASS | `2026-06-19T11:42:47+08:00` | runtime smoke 验证已通过 |
| inline fallback 授权 | WAIVED | 当前对话 | 用户逐步提供 Windows gateway 状态、HMAC env 和授权 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP7 结论合法 | PASS | 本文件 | `PASS` |
| 可进入 CP8 | PASS | `process/release/RELEASE-CONTEXT-CR097.yaml` | 准备发布就绪人工终验 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| CP7 result | `process/checks/CP7-CR097-REAL-QMT-READONLY-RUNTIME-SMOKE-VERIFICATION-DONE.md` | PASS | 已生成 |
| Release context | `process/release/RELEASE-CONTEXT-CR097.yaml` | PASS | 已生成 |

## 结论

- 结论：`PASS`
- 阻断项：无
- 豁免项：未启动 meta-qa，按本轮 runtime authorization 使用 host-orchestrator inline-fallback
- 剩余风险：`R-CR097-01` 本次模拟新账户为空持仓且为非交易日；不覆盖非空持仓、交易日、实盘账户、订单 / 成交 / 资金查询或交易写接口
- 下一步：生成 CP8 自动预检和人工审查稿
