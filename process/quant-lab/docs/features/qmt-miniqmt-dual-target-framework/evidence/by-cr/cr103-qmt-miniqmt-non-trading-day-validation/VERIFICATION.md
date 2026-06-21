---
status: complete
version: "1.0"
story_id: "CR103"
story_slug: "qmt-miniqmt-non-trading-day-validation"
feature_id: "qmt-miniqmt-runtime-boundary"
validation_mode: "mixed"
verification_result: "PASS_WITH_RISK"
source_story: "process/changes/CR-103-QMT-MINIQMT-NON-TRADING-DAY-VALIDATION-EXPERIMENT-2026-06-21.md"
source_implementation: "tests/test_cr019_qmt_endpoint_matrix.py"
created_by: "host-orchestrator"
created_at: "2026-06-21T13:31:01+08:00"
updated_at: "2026-06-21T13:31:01+08:00"
---

# Verification: CR103 QMT/MiniQMT Non-Trading-Day Validation

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | PASS_WITH_RISK |
| validation_mode | mixed |
| 是否可进入下一阶段 | yes |
| 需要路由 | host-orchestrator / human for CR104 |
| CP7 证据 | `process/checks/CP7-CR103-QMT-MINIQMT-NON-TRADING-DAY-VALIDATION.md` |

## 2. 验证范围

| 项 | 内容 |
|---|---|
| 验证范围 | QMT direct-run target contract、MiniQMT readonly adapter contract、QMT endpoint matrix、gateway allowlist、HMAC / session gate、CR099 redacted evidence checker、CR089 package skeleton checker。 |
| 非范围 | 真实 QMT terminal 启动、MiniQMT/XtQuant/gateway runtime、env/credential/account read、query_positions 真实调用、submit/cancel/simulation/live、交易日行情/账户数据。 |
| 上游设计 | `docs/qmt/CR101-CROSS-PLATFORM-STRATEGY-DELIVERY-ADAPTER-REALIGNMENT-HLD.md`、`docs/qmt/CR101-VALIDATION-AND-FOLLOW-UP-GATES.md`、`docs/qmt/CR099-QMT-RUNNER-REAL-READONLY-SMOKE-HLD.md` |
| 已接受风险 | aggregate / runtime evidence 尚未真实运行；交易日验证后置 CR104。 |

## 3. 验证对象清单

| 对象 | 类型 | 验证方式 | 是否阻塞 | 证据 |
|---|---|---|---|---|
| `packages/qmt_interface_smoke/0.1.0` | package / docs | local checker | yes | checker PASS |
| `trading/strategy_runner/*` | code contract | pytest fixture | yes | CR091/CR098 tests PASS |
| `trading/qmt_client.py` / `trading/qmt_endpoint_matrix.py` | code contract | endpoint matrix pytest | yes | CR019 tests PASS |
| `trading/qmt_gateway_session.py` | code contract | session gate pytest | yes | CR020 test PASS |
| `scripts/check_cr099_redacted_evidence.py` | guardrail-validator | pytest fixture | yes | CR099 tests PASS |
| `tests/test_cr019_qmt_endpoint_matrix.py` | test contract | patched + regression | yes | PASS after fix |

## 4. 验证追踪矩阵

| Scenario | Design Contract | Test / Check | Status | Risk |
|---|---|---|---|---|
| QMT direct-run package target offline ready | only implemented target is `qmt_terminal_direct` | CR091 / CR101 package tests | PASS | 不证明真实 QMT ready |
| MiniQMT remains readonly adapter, not runner host | adapter allowlist health/capabilities/query_positions only | CR098 / CR099 tests | PASS | 不证明真实 gateway ready |
| endpoint matrix blocks order/simulation/live | later-gated endpoints fail closed | CR019 endpoint matrix tests | PASS | positions 无 transport 返回 transport unavailable |
| session gate blocks query when not ready | no adapter call unless session ready | CR020 session tests | PASS | 真实 session 需 CR104 |
| redacted evidence rejects raw payload / forbidden counters | no raw account/log/order evidence | CR099 checker tests | PASS | 真实 evidence 需 CR104 |

## 5. 自动化验证结果

| Command ID | 命令 / 检查 | 结果 | 说明 |
|---|---|---|---|
| CMD-01 | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr089_qmt_interface_smoke_package.py --package-root packages/qmt_interface_smoke/0.1.0` | PASS | `passed=true`, `errors=[]` |
| CMD-02 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr089_qmt_interface_smoke_package.py tests/test_cr091_strategy_runner_contracts.py tests/test_cr098_runner_readonly_integration.py tests/test_cr099_runner_real_readonly_smoke_contract.py tests/test_cr019_qmt_gateway_run_gates.py tests/test_cr019_qmt_endpoint_matrix.py tests/test_cr019_qmt_gateway_fallback.py tests/test_cr019_qmt_cside_client_cli.py tests/test_cr020_server_qmt_login_session.py tests/test_cr015_qmt_environment_boundary.py tests/test_cr015_qmt_adapter_contract.py tests/test_cr016_simulation_order_enable_gate.py` | PASS | `125 passed in 0.69s` |

## 6. 问题清单

| ID | 等级 | 问题 | 影响 | 建议处理 | Owner | 状态 |
|---|---|---|---|---|---|---|
| CR103-Q-001 | MEDIUM | 旧测试将 `positions` 与其他 later-gated endpoint 同口径断言为 `per_run_authorization_missing`，但当前 CR020/ADR-092 已将 `query_positions` 作为唯一真实只读 REST endpoint，未配置 transport 时应 `transport_unavailable`。 | 阻断非交易日验证回归。 | 修正测试断言并重跑验证集。 | host-orchestrator | RESOLVED |

## 7. 剩余风险

| Risk ID | 风险 | 等级 | 是否接受 | 后续处理 |
|---|---|---|---|---|
| CR103-R-001 | 本轮验证不证明真实 QMT terminal、MiniQMT gateway、XtQuant 或 Windows runtime 可用。 | HIGH | yes, bounded to CR103 | CR104 交易日 runtime gate |
| CR103-R-002 | 本轮没有真实 query_positions、health、capabilities runtime evidence。 | HIGH | yes, bounded to CR103 | CR104 手工/受控 evidence |
| CR103-R-003 | order-write / simulation / live 仍未设计和授权。 | HIGH | yes | FU-CR101-001 / later CR |

## 8. 阶段决策

| 结论 | 路由 | 条件 / 说明 |
|---|---|---|
| PASS_WITH_RISK | host-orchestrator / human | 非交易日可验证内容完成；真实 runtime 与交易日内容进入 CR104。 |
