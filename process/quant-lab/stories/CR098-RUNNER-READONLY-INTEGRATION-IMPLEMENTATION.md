---
story_id: "CR098-RUNNER-READONLY-INTEGRATION"
story_slug: "RUNNER-READONLY-INTEGRATION"
status: "ready-for-verification"
owner: "host-orchestrator"
created_at: "2026-06-19T12:18:02+08:00"
scope: "offline implementation and fixture transport tests only"
---

# CR098 Runner Readonly Integration Implementation

## 实现前置检查

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP5 已批准 | PASS | `process/checkpoints/CP5-CR098-RUNNER-READONLY-INTEGRATION-READINESS.md` | 用户回复“同意”，接受 DQ-CP5-CR098-01..05。 |
| 设计证据可读 | PASS | `process/stories/CR098-RUNNER-READONLY-INTEGRATION-LLD.md` | 状态更新为 `approved-for-offline-implementation`。 |
| 文件所有权 | PASS | CP5 Decision Brief / LLD | 仅修改 runner adapter、evidence、qmt client 可选授权参数、测试和过程证据。 |
| 禁止范围 | PASS | 本轮命令清单 | 未读取 HMAC env、Windows `.env`、账户原文；未启动 gateway；未执行 runner runtime；未访问 NAS。 |

## 实现对象清单

| 对象 | 路径 | 类型 | 验证方式 |
|---|---|---|---|
| readonly runtime config / facade | `trading/strategy_runner/readonly_gateway.py` | code | CR098 pytest |
| qmt client authorization_ref passthrough | `trading/qmt_client.py` | code | CR020 runtime / HMAC 回归 |
| evidence summary extension | `trading/strategy_runner/evidence.py` | code | CR098 / CR091 pytest |
| package export | `trading/strategy_runner/__init__.py` | package API | import tests |
| CR098 contract tests | `tests/test_cr098_runner_readonly_integration.py` | tests | pytest |

## 设计契约映射

| 设计契约 | 实现位置 | 验证 |
|---|---|---|
| 默认 fake/offline，不打开 socket | `ReadonlyGatewayClient` 默认无 `QmtClient` 时走 `FakeReadonlyQmtTransport` | `test_cr098_default_gateway_client_remains_fake_and_offline` |
| real adapter 只通过 injected `QmtClient` / transport | `ReadonlyGatewayClient(client=..., runtime_config=...)` 与 `from_transport()` | `test_cr098_qmt_client_fixture_path_normalizes_readonly_results` |
| 未显式 runtime authorization fail closed | `_call_qmt_client()` 检查 `ReadonlyGatewayRuntimeConfig.enabled` | `test_cr098_qmt_client_path_requires_explicit_runtime_authorization` |
| endpoint allowlist | `ALLOWED_READONLY_ENDPOINTS` | `test_cr098_readonly_gateway_still_blocks_order_like_endpoints` |
| redacted evidence summary | `EvidenceSummary` 新增 readonly fields | `test_cr098_evidence_records_redacted_readonly_summary` |
| sensitive/raw payload 拒收 | `build_evidence_summary()` 对 readonly payload 调用 `assert_redacted()` | `test_cr098_evidence_blocks_sensitive_or_raw_readonly_payload` |
| runner 不 import runtime-only boundary | source static test | `test_cr098_readonly_gateway_source_does_not_import_runtime_boundaries` |

## 单元测试与 Fixture 计划

| Fixture / 测试 | 覆盖点 | 状态 |
|---|---|---|
| `RecordingTransportFixture` | injected `QmtClient` / transport，不访问真实 gateway | PASS |
| `HeaderProviderFixture` | auth header provider 输出 fixture headers，不读取 secret | PASS |
| CR091 strategy package fixture | adapter_result 与 evidence 组合 | PASS |
| CR020 runtime / HMAC 回归 | qmt client 兼容性 | PASS |

## 最小实现切片

| Slice | 输出 | 局部验证 | 回滚点 |
|---|---|---|---|
| S1 qmt client auth passthrough | `trading/qmt_client.py` | CR020 runtime / HMAC tests | 删除新增 optional 参数 |
| S2 readonly facade | `readonly_gateway.py` / export | CR098 adapter tests | 恢复 fake-only wrapper |
| S3 evidence summary | `evidence.py` | CR098 / CR091 evidence tests | 删除新增 readonly summary fields |
| S4 CP6 证据 | 本文件 / CP6 check | human-gate / CR tracking / diff check | 回退过程证据 |

## 平台差异处理

N/A。CR098 CP6 不安装 Claude / Codex / OpenClaw 平台产物，不生成 agent / skill，不触发平台目录安装结构。

## 验证结果

| 命令 | 结果 | 说明 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr098_runner_readonly_integration.py` | PASS，`7 passed in 0.11s` | CR098 新 contract tests。 |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr091_strategy_runner_contracts.py` | PASS，`13 passed in 0.16s` | CR091 runner 回归。 |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr020_runtime_manual_validation.py` | PASS，`6 passed in 0.12s` | CR020 runtime transport 回归。 |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr020_hmac_pairing_allowlist_scope.py` | PASS，`15 passed in 0.11s` | HMAC / allowlist / scope 回归。 |
| `PYTHONPYCACHEPREFIX=/tmp/cr098-cp6-pycompile uv run --python 3.11 python -m py_compile ...` | PASS | 目标文件编译通过。 |

## 未覆盖项

- 未执行真实 Windows gateway runner smoke：CP5 明确不授权，后续 CP7 如需真实 runtime 必须逐 run 授权。
- 未读取 `/home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template`、Windows `.env`、账户原文或持仓原文。
- 未访问 NAS、provider/lake/publish、submit/cancel、simulation/live。

## 设计缺口反馈

- `gateway_socket_open` 在未来真实 runtime evidence 中需要作为“已授权只读 transport activity”单独解释，不能简单复用默认离线 forbidden=0 语义。
- 当前模拟新账户空持仓和非交易日仍不能证明非空持仓脱敏路径；应由后续非空/交易日 follow-up 独立补测。

## 后续交接

- CP7 可先执行离线验证与 review，不需要真实 gateway。
- 如用户要求真实 runner smoke，必须重新明确授权 env path、host/port、runtime authorization ref 和 evidence 输出路径。
