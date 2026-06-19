---
verification_id: "cr098-runner-readonly-integration"
cr_id: "CR-098"
status: "PASS_WITH_RISK"
owner: "host-orchestrator"
created_at: "2026-06-19T12:22:28+08:00"
validation_mode: "mixed"
---

# CR098 Runner Readonly Integration Verification

## 1. 验证范围

| 范围 | 内容 | 结论 |
|---|---|---|
| In Scope | runner readonly adapter fake default、injected `QmtClient` fixture path、evidence summary、contract tests、CR091/CR020 回归 | PASS |
| Out of Scope | HMAC env 读取、Windows gateway 启动、真实 runner runtime、账户原文、NAS、交易写、simulation/live、provider/lake/publish | NOT_AUTHORIZED |

## 2. 验证对象清单

| 对象 | 路径 | 验证方式 | 结果 |
|---|---|---|---|
| readonly facade | `trading/strategy_runner/readonly_gateway.py` | CR098 pytest + static scan | PASS |
| qmt client auth passthrough | `trading/qmt_client.py` | CR020 runtime / HMAC 回归 | PASS |
| evidence summary | `trading/strategy_runner/evidence.py` | CR098 / CR091 pytest | PASS |
| package exports | `trading/strategy_runner/__init__.py` | import / pytest | PASS |
| tests | `tests/test_cr098_runner_readonly_integration.py` | pytest | PASS |
| CP6 evidence | `process/stories/CR098-RUNNER-READONLY-INTEGRATION-IMPLEMENTATION.md` | 人工/语义审查 | PASS |

## 3. 验证追踪矩阵

| Design Contract | Implementation | Test / Check | Status | Risk |
|---|---|---|---|---|
| 默认 fake/offline | `ReadonlyGatewayClient` 默认 fake transport | `test_cr098_default_gateway_client_remains_fake_and_offline` | PASS | 低 |
| real adapter 只通过 injected `QmtClient` | `ReadonlyGatewayRuntimeConfig` + `client` injection | `test_cr098_qmt_client_fixture_path_normalizes_readonly_results` | PASS | 中 |
| 未授权 runtime fail closed | `_call_qmt_client()` authorization guard | `test_cr098_qmt_client_path_requires_explicit_runtime_authorization` | PASS | 高 |
| endpoint allowlist | `ALLOWED_READONLY_ENDPOINTS` | `test_cr098_readonly_gateway_still_blocks_order_like_endpoints` | PASS | 高 |
| evidence 脱敏 | `EvidenceSummary` readonly fields + `assert_redacted` | `test_cr098_evidence_records_redacted_readonly_summary` / sensitive payload test | PASS | 高 |
| CR091 兼容 | evidence / fake gateway 回归 | `tests/test_cr091_strategy_runner_contracts.py` | PASS | 中 |
| CR020 兼容 | qmt client auth/header 回归 | CR020 runtime / HMAC tests | PASS | 中 |

## 4. 分层验证计划执行

| 层级 | 命令 / 方式 | 结果 |
|---|---|---|
| workspace route | `uv run --python 3.11 meta-flow workspace check --project-root .` | PASS |
| contract tests | `pytest -q tests/test_cr098_runner_readonly_integration.py tests/test_cr091_strategy_runner_contracts.py tests/test_cr020_runtime_manual_validation.py tests/test_cr020_hmac_pairing_allowlist_scope.py` | PASS，41 passed |
| syntax | `python -m py_compile ...` | PASS |
| CR tracking | `meta-flow check cr-tracking --strict-warnings` | OK |
| whitespace | `git diff --check` / artifact `diff --check` | PASS |
| runtime | N/A | 未授权 |

## 5. 问题与剩余风险

| 风险 ID | 等级 | 状态 | 说明 | 后续 |
|---|---|---|---|---|
| R-CR098-01 | HIGH | accepted-for-cp7 | CP7 未证明真实 runner runtime smoke | CP8 风险接受；如需运行另起逐 run 授权 |
| R-CR098-02 | MEDIUM | accepted-for-cp7 | 当前模拟新账户空持仓、非交易日，未覆盖非空脱敏 | 后续非空 / 交易日 readonly follow-up |
| R-CR098-03 | MEDIUM | accepted-for-cp7 | `gateway_socket_open` 在真实授权 runtime 中需重新分类，不能按离线 forbidden=0 简单判断 | CP7 runtime smoke 设计时单独处理 |

## 6. 阶段决策

- 结论：`PASS_WITH_RISK`
- 阻断项：无
- 需回修项：无
- 可进入：CP8 delivery readiness / risk acceptance
- 不授权项：真实 runtime、secret/env、账户原文、NAS、交易写、simulation/live、provider/lake/publish
