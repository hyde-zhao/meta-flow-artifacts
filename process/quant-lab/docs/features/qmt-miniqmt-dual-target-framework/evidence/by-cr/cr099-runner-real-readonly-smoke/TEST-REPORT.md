---
test_report_id: "cr099-runner-real-readonly-smoke"
cr_id: "CR-099"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-19T16:20:16+08:00"
---

# CR099 Runner Real Readonly Smoke Test Report

## 测试范围

| 范围 | 状态 | 说明 |
|---|---|---|
| CR099 contract / evidence checker / collector tests | PASS | 9 项通过 |
| CR098 runner readonly regression | PASS | 7 项通过 |
| py_compile | PASS | checker 和 tests 编译通过 |
| CLI help | PASS | checker 命令可用，未读取 evidence/env |
| 真实 Windows runner smoke | PASS | 用户刷新 gateway session 后，health / capabilities / query_positions_readonly 通过，redacted evidence schema PASS |

## 命令结果

| # | 命令 | Exit | 结果 |
|---|---|---:|---|
| 1 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr099_runner_real_readonly_smoke_contract.py` | 0 | `9 passed` |
| 2 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr099_runner_real_readonly_smoke_contract.py tests/test_cr098_runner_readonly_integration.py` | 0 | `16 passed` |
| 3 | `PYTHONPYCACHEPREFIX=/tmp/cr099-cp6-pycompile uv run --python 3.11 python -m py_compile scripts/check_cr099_redacted_evidence.py tests/test_cr099_runner_real_readonly_smoke_contract.py` | 0 | PASS |
| 4 | `uv run --python 3.11 python scripts/check_cr099_redacted_evidence.py --help` | 0 | PASS |
| 5 | `uv run --python 3.11 meta-flow workspace check` | 0 | `process_link_health: ok` |
| 6 | `uv run --python 3.11 meta-flow check cr-tracking --project-root . --strict-warnings` | 0 | OK |
| 7 | `git diff --check` | 0 | PASS |
| 8 | `git -C /home/hyde/workspace/meta-flow-artifacts diff --check` | 0 | PASS |
| 9 | `uv run --python 3.11 python scripts/collect_cr099_runner_readonly_smoke.py --env-file /home/hyde/.quant-lab/runtime/qmt/cr097/client.env.template --base-url http://172.30.32.1:18765 --authorization-ref cr099-runner-readonly-smoke-20260619-sim --run-id cr099-runner-readonly-smoke-20260619-sim-001 ...` | 0 | PASS，evidence `overall_status=pass` |
| 10 | `uv run --python 3.11 python scripts/check_cr099_redacted_evidence.py --evidence /home/hyde/.quant-lab/evidence/qmt/cr099/redacted/cr099-runner-readonly-smoke-20260619-sim-001/evidence.json --json` | 0 | PASS，errors=0，warnings=0 |

## 覆盖缺口

| 缺口 | 原因 | 风险 | 后续 |
|---|---|---|---|
| 非空持仓脱敏未覆盖 | 当前授权 run 返回空持仓 | MEDIUM | 后续非空 / 交易日复测独立授权 |
| 全量 pytest 未运行 | 当前 CR099 影响面限定 checker / runner readonly regression | LOW | 发布前可按需扩展 |

## 阶段结论

测试层面满足 CR099 CP7 runtime 验证要求，结论为 `PASS`。当前可进入 CP8 delivery readiness；非空持仓 / 交易日复测保留为独立 follow-up 风险。
