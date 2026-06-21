---
feature_id: "cr098-runner-readonly-integration"
cr_id: "CR-098"
status: "implemented-pending-cp7"
owner: "host-orchestrator"
created_at: "2026-06-19T12:07:32+08:00"
source_hld: "docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md"
---

# CR098 Runner Readonly Integration Feature Design

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-19 | host-orchestrator | 初版 CP5 Feature Design，冻结 runner readonly gateway adapter、runtime authorization 和 evidence 合同。 |
| 0.2 | 2026-06-19 | host-orchestrator | CP5 approved 后完成 CP6 offline implementation，等待 CP7 验证。 |

## 1. Feature 边界

CR098 只把 CR091 runner 与 CR097 已验证的 Windows gateway readonly 能力衔接到同一 runner evidence 链路。默认路径仍为 fake/offline；真实 REST transport 只能在后续逐 run runtime authorization 中显式构造。

## 2. 能力地图

| 能力 | 目标 | 首版实现 |
|---|---|---|
| readonly adapter facade | runner 统一调用 health / capabilities / query_positions | 扩展 `trading/strategy_runner/readonly_gateway.py` |
| real transport injection | 后续授权 run 可注入 `QmtClient` + `StdlibQmtRestTransport` | 只定义显式 factory / config contract |
| endpoint allowlist | 阻断 submit / cancel / simulation / live | 保持 `ALLOWED_READONLY_ENDPOINTS`，新增 tests |
| redacted evidence | 输出 CR098 摘要和 forbidden counters | 扩展 `trading/strategy_runner/evidence.py` |
| runtime authorization manifest | 把设计批准和运行授权分离 | 新增参数 / evidence 字段，不默认读取 env |

## 3. 领域对象

| 对象 | 字段 | 说明 |
|---|---|---|
| `ReadonlyGatewayRuntimeConfig` | `base_url_ref`、`authorization_ref`、`runtime_env_ref`、`timeout_seconds` | 只保存 ref/hash，不保存 secret |
| `ReadonlyGatewayResult` | `endpoint`、`status`、`payload`、`reason_code`、`operation_counters`、`transport_kind` | 统一 fake/real 结果 |
| `EvidenceSummary` | `readonly_health_status`、`readonly_capabilities_status`、`position_count`、`positions_digest`、`items_redacted_count` | CR098 runtime summary |

## 4. 依赖方向

`trading/strategy_runner/*` 可以依赖 `trading.qmt_client` 的 typed client 和 protocol；只有显式 runtime factory 可以引用 `trading.qmt_runtime.StdlibQmtRestTransport`。runner 不得 import `xtquant`、不得读取 `.env`、不得启动 Windows gateway。

## 5. 失败行为

| 失败 | 行为 |
|---|---|
| 未传 runtime authorization | 使用 fake/offline 或返回 blocked，真实 HTTP call 为 0 |
| env path 未授权 | blocked，`credential_read=0` |
| auth header 缺失 / 错误 | blocked，不输出 raw response |
| gateway unavailable | `transport_error` / blocked evidence |
| raw payload emitted | evidence blocked |
| forbidden counter 非 0 | evidence blocked |

## 6. 验收标准

- 默认测试不打开 socket、不读 env、不查询账户。
- fake transport 覆盖 health / capabilities / query_positions。
- real transport fixture 使用注入 opener，不访问真实 gateway。
- order / cancel / simulation / live endpoint 均 blocked。
- evidence 不包含 token、secret、account、raw positions、orders 或 qmt logs。
