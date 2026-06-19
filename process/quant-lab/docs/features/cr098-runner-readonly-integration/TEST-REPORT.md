---
test_report_id: "cr098-runner-readonly-integration"
cr_id: "CR-098"
status: "PASS_WITH_RISK"
owner: "host-orchestrator"
created_at: "2026-06-19T12:22:28+08:00"
---

# CR098 Runner Readonly Integration Test Report

## 测试范围

| 范围 | 状态 | 说明 |
|---|---|---|
| CR098 新增 contract tests | PASS | 7 项通过 |
| CR091 runner 回归 | PASS | 13 项通过 |
| CR020 runtime transport 回归 | PASS | 6 项通过 |
| CR020 HMAC / allowlist / scope 回归 | PASS | 15 项通过 |
| 真实 Windows runner smoke | NOT_RUN | 未授权 |

## 命令结果

| # | 命令 | Exit | 结果 |
|---|---|---:|---|
| 1 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr098_runner_readonly_integration.py tests/test_cr091_strategy_runner_contracts.py tests/test_cr020_runtime_manual_validation.py tests/test_cr020_hmac_pairing_allowlist_scope.py` | 0 | `41 passed in 0.34s` |
| 2 | `PYTHONPYCACHEPREFIX=/tmp/cr098-cp6-pycompile-final uv run --python 3.11 python -m py_compile trading/qmt_client.py trading/strategy_runner/readonly_gateway.py trading/strategy_runner/evidence.py trading/strategy_runner/__init__.py tests/test_cr098_runner_readonly_integration.py` | 0 | PASS |
| 3 | `uv run --python 3.11 meta-flow workspace check --project-root .` | 0 | `process_link_health: ok` |
| 4 | `uv run --python 3.11 meta-flow check cr-tracking --project-root . --strict-warnings` | 0 | OK |
| 5 | `git diff --check` | 0 | PASS |
| 6 | `git -C /home/hyde/workspace/meta-flow-artifacts diff --check` | 0 | PASS |

## 覆盖缺口

| 缺口 | 原因 | 风险 | 后续 |
|---|---|---|---|
| 真实 runner -> Windows gateway smoke 未执行 | CP5 不授权 runtime | HIGH | 逐 run 授权后执行 |
| 非空持仓脱敏未覆盖 | 当前模拟新账户空持仓，且今天为非交易日 | MEDIUM | 后续交易日 / 非空账户复测 |
| 全量 pytest 未运行 | 当前 CR098 影响面限定 runner / qmt client / evidence | LOW | 发布前可按需扩展 |

## 阶段结论

测试层面满足 CR098 CP7 离线验证要求，结论为 `PASS_WITH_RISK`。
