---
story_id: "CR098-RUNNER-READONLY-INTEGRATION"
cr_id: "CR-098"
status: "draft-pending-cp5-approval"
lld_policy: "technical-note"
owner: "host-orchestrator"
created_at: "2026-06-19T12:07:32+08:00"
source_hld: "docs/qmt/CR098-QMT-RUNNER-READONLY-GATEWAY-INTEGRATION-HLD.md"
---

# CR098 Runner Readonly Integration LLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-06-19 | host-orchestrator | 初版 LLD，冻结文件影响范围、接口、失败行为、测试和运行授权边界。 |

## 1. 实现目标

把 `ReadonlyGatewayClient` 从 CR091 fake-only wrapper 升级为 fake/real 可注入 readonly facade。real 路径只复用 `QmtClient` typed contract，不隐式读取 env，不启动 gateway，不 import XtQuant。

## 2. 文件影响范围

| 文件 | 变更 |
|---|---|
| `trading/strategy_runner/readonly_gateway.py` | 新增 runtime config / transport_kind / explicit factory；保持 fake default |
| `trading/strategy_runner/evidence.py` | 扩展 evidence summary，支持 CR098 health / capabilities / query_positions 摘要 |
| `trading/strategy_runner/__init__.py` | 导出新增 contract |
| `tests/test_cr098_runner_readonly_integration.py` | 新增 contract / redaction / fixture tests |
| `tests/test_cr091_strategy_runner_contracts.py` | 保持回归通过，必要时只更新兼容断言 |

## 3. 接口合同

```python
@dataclass(frozen=True, slots=True)
class ReadonlyGatewayRuntimeConfig:
    base_url: str
    authorization_ref: str
    runtime_env_ref: str
    timeout_seconds: int = 10

class ReadonlyGatewayClient:
    def health(self, *, run_id: str) -> ReadonlyGatewayResult: ...
    def capabilities(self, *, run_id: str) -> ReadonlyGatewayResult: ...
    def query_positions(self, *, run_id: str) -> ReadonlyGatewayResult: ...
```

`runtime_env_ref` 只能是脱敏 ref，例如 `cr098-client-env:sha256:<digest>`；不得保存路径中的 secret 或文件内容。

## 4. 数据映射

| 来源 | 目标字段 | 规则 |
|---|---|---|
| `QmtResponse.status` | `ReadonlyGatewayResult.status` | ok 保持 ok；blocked / auth_error / transport_error 归一化 |
| `QmtResponse.payload` | `ReadonlyGatewayResult.payload` | 只保留 redacted summary |
| `QmtResponse.counters` | `operation_counters` | 原样纳入 forbidden counter 评估 |
| positions payload | evidence `position_count` / digest / redacted count | 不保留 raw rows |

## 5. 失败路径

- `authorization_ref` 空：blocked。
- `auth_header_provider` 缺失：blocked。
- transport unavailable：blocked / transport_error。
- response 含 raw 或 sensitive key：blocked。
- forbidden counter 非 0：blocked。

## 6. 测试设计

见 `docs/features/cr098-runner-readonly-integration/TEST-PLAN.md`。默认测试使用 fake transport 和 fake opener；真实 gateway 只在后续逐 run 授权下运行。

## 7. 发布与回滚

回滚方式是恢复 `ReadonlyGatewayClient` fake-only 行为并删除 CR098 新 tests / evidence schema 扩展。回滚不得删除 CR091 已交付 runner 合同。

## 8. Gotchas

- `gateway_socket_open` 在真实 runtime smoke 中可能为正数；默认离线 tests 必须为 0。CP7 runtime evidence 需要区分 authorized readonly transport counter 与 forbidden order/account-write counters。
- HMAC pass 只证明调用方身份，不证明交易授权。
- 空持仓和非交易日不能证明非空脱敏路径，需后续 CR097-FU-01 或 CR098 follow-up 补测。
