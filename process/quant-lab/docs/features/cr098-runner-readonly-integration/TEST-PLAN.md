---
test_plan_id: "cr098-runner-readonly-integration"
cr_id: "CR-098"
status: "implemented-pending-cp7"
owner: "host-orchestrator"
created_at: "2026-06-19T12:07:32+08:00"
---

# CR098 Runner Readonly Integration Test Plan

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-19 | host-orchestrator | 初版测试计划，覆盖离线 contract、real transport fixture、redaction 和可选 runtime smoke。 |
| 0.2 | 2026-06-19 | host-orchestrator | CP6 已新增离线/fixture 测试并通过，真实 runtime smoke 仍未授权。 |

## 1. 测试策略

默认验证只跑离线和 fixture。真实 Windows gateway runner smoke 只作为 CP7 可选项，必须用户逐 run 授权后才执行。

## 2. 测试分层

| 层级 | 测试对象 | 命令 / 方式 | 是否需要真实 QMT |
|---|---|---|---|
| unit | adapter allowlist / fake default | `uv run --python 3.11 pytest -q tests/test_cr098_runner_readonly_integration.py` | 否 |
| unit | evidence redaction / forbidden counters | 同上 | 否 |
| fixture integration | `QmtClient` + injected `StdlibQmtRestTransport` fake opener | 同上 | 否 |
| static safety | forbidden import / env read scan | pytest 或 `rg` 断言 | 否 |
| optional runtime | runner -> Windows gateway readonly smoke | 后续授权命令 | 是，逐 run 授权 |

## 3. 必测场景

| 场景 ID | Given | When | Then |
|---|---|---|---|
| TS-CR098-01 | 默认 runner readonly client | 调用 health / capabilities / query_positions | 返回 ok，fake flag 为 true，forbidden counters 全 0 |
| TS-CR098-02 | 未授权 runtime config | 尝试 real adapter | blocked 或 fake-only，credential_read / gateway_socket_open 为 0 |
| TS-CR098-03 | injected HTTP fixture | 调用 health / capabilities / query_positions | 结果进入统一 `ReadonlyGatewayResult` |
| TS-CR098-04 | submit / cancel / simulation / live endpoint | 调用 facade | blocked_scope_denied |
| TS-CR098-05 | readonly payload 含 raw / sensitive 字段 | 写 evidence | 抛出 redaction error 或 evidence status blocked |
| TS-CR098-06 | gateway_socket_open / account_query counter 非 0 | 汇总 evidence | status blocked |

## 4. 可选 runtime smoke 接受条件

可选 runtime smoke 只在 CP7 之后、用户逐 run 授权时执行。接受条件：

- gateway health ok；
- capabilities 包含 readonly endpoint；
- query_positions_readonly 返回 redacted summary；
- `raw_payload_emitted=false`；
- 账户为模拟账户或用户明确声明的只读账户；
- evidence 写入 `/home/hyde/.quant-lab/evidence/qmt/cr098/redacted/`；
- 不打印 secret、账号原文、持仓原文、资金、委托、成交或日志。

## 5. 拒收规则

- 任何 `.env`、HMAC secret、account id 原文、raw positions、orders、trades 或 qmt logs 进入 Git / process / docs / evidence：FAIL。
- 默认 pytest 打开真实 socket 或读取 env：FAIL。
- runner import `xtquant`：FAIL。
- submit / cancel / buy / sell / simulation / live 能穿透 facade：FAIL。
