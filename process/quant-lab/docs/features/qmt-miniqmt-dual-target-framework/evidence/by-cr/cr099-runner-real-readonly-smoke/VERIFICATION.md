---
verification_id: "cr099-runner-real-readonly-smoke"
cr_id: "CR-099"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-19T16:20:16+08:00"
validation_mode: "mixed"
---

# CR099 Runner Real Readonly Smoke Verification

## 1. 验证范围

| 范围 | 内容 | 结论 |
|---|---|---|
| In Scope | CR099 run contract checker、redacted evidence checker、collector、negative fixtures、CR098 runner readonly regression、runtime evidence、process evidence | PASS |
| Authorized Runtime | Windows gateway 已由用户手动启动；WSL client-side runner smoke 调用 health / capabilities / query_positions_readonly | PASS |
| Out of Scope | Windows `.env` 读取、账户原文、NAS、交易写、simulation/live、provider/lake/publish、非空持仓 / 交易日复测 | NOT_AUTHORIZED |

## 2. 验证对象清单

| 对象 | 路径 | 验证方式 | 结果 |
|---|---|---|---|
| CR099 checker | `scripts/check_cr099_redacted_evidence.py` | CR099 pytest、py_compile、CLI help、static import scan | PASS |
| CR099 tests | `tests/test_cr099_runner_real_readonly_smoke_contract.py` | pytest | PASS |
| CR099 collector | `scripts/collect_cr099_runner_readonly_smoke.py` | pytest、py_compile、runtime rerun | PASS |
| Runtime evidence | `/home/hyde/.quant-lab/evidence/qmt/cr099/redacted/cr099-runner-readonly-smoke-20260619-sim-001/evidence.json` | CR099 checker、脱敏摘要审查 | PASS |
| CR098 regression | `tests/test_cr098_runner_readonly_integration.py` | pytest | PASS |
| CP6 evidence | `process/stories/CR099-RUNNER-REAL-READONLY-SMOKE-IMPLEMENTATION.md` | 人工 / 语义审查 | PASS |
| process state | `process/STATE.md` / `process/changes/CR-INDEX.yaml` | cr-tracking | PASS |

## 3. 验证追踪矩阵

| Design Contract | Implementation | Test / Check | Status | Risk |
|---|---|---|---|---|
| run contract 必填字段 | `check_run_contract()` | `test_cr099_run_contract_accepts_minimal_authorized_reference` | PASS | 中 |
| 只允许 readonly allowlist | `ALLOWED_ENDPOINTS` | `test_cr099_run_contract_rejects_order_endpoint` | PASS | 高 |
| contract 不保存 secret / `.env` | forbidden marker validation | `test_cr099_run_contract_rejects_secret_like_env_ref` | PASS | 高 |
| evidence 覆盖 3 endpoint | `check_redacted_evidence()` | `test_cr099_redacted_evidence_accepts_zero_position_summary` | PASS | 中 |
| forbidden counters 必须全 0 | `REQUIRED_FORBIDDEN_COUNTERS` | `test_cr099_redacted_evidence_rejects_nonzero_forbidden_counter` | PASS | 高 |
| raw payload 不得输出 | raw payload false validation | `test_cr099_redacted_evidence_rejects_raw_payload_marker` | PASS | 高 |
| checker 不触碰 runtime / network | AST import static check | `test_cr099_checker_does_not_import_runtime_or_network_modules` | PASS | 高 |
| collector 输出 CR099 evidence | `build_cr099_evidence()` / collector CLI | `test_cr099_collector_builds_checker_accepted_redacted_evidence` + runtime rerun | PASS | 中 |
| CR098 兼容 | existing runner readonly integration tests | `tests/test_cr098_runner_readonly_integration.py` | PASS | 中 |

## 4. 分层验证计划执行

| 层级 | 命令 / 方式 | 结果 |
|---|---|---|
| workspace route | `uv run --python 3.11 meta-flow workspace check` | PASS |
| contract / schema / collector tests | `pytest -q tests/test_cr099_runner_real_readonly_smoke_contract.py` | PASS，9 passed |
| regression | `pytest -q tests/test_cr099_runner_real_readonly_smoke_contract.py tests/test_cr098_runner_readonly_integration.py` | PASS，16 passed |
| syntax | `python -m py_compile scripts/check_cr099_redacted_evidence.py tests/test_cr099_runner_real_readonly_smoke_contract.py` | PASS |
| CLI | `uv run --python 3.11 python scripts/check_cr099_redacted_evidence.py --help` | PASS |
| runtime rerun | `scripts/collect_cr099_runner_readonly_smoke.py ... --base-url http://172.30.32.1:18765 ...` | PASS，health / capabilities / query_positions_readonly 均通过 |
| runtime evidence checker | `scripts/check_cr099_redacted_evidence.py --evidence /home/hyde/.quant-lab/evidence/qmt/cr099/redacted/cr099-runner-readonly-smoke-20260619-sim-001/evidence.json --json` | PASS，errors=0，warnings=0 |
| CR tracking | `meta-flow check cr-tracking --strict-warnings` | OK |
| whitespace | `git diff --check` / artifact `diff --check` | PASS |

## 5. 问题与剩余风险

| 风险 ID | 等级 | 状态 | 说明 | 后续 |
|---|---|---|---|---|
| R-CR099-01 | HIGH | resolved-runtime-pass | CP7 已证明 runner -> Windows gateway readonly smoke | 证据见 `CP7-CR099-RUNTIME-AUTHORIZED-SMOKE-RERUN-PASS-2026-06-19.md` |
| R-CR099-02 | HIGH | controlled | 本轮仅读取用户授权的 WSL client env 引用；未读取 Windows `.env` 或输出 secret | CP8 记录授权边界和证据路径 |
| R-CR099-03 | MEDIUM | residual-follow-up | 当前 evidence 为空持仓路径，不证明非空持仓脱敏路径 | 非空 / 交易日复测独立 follow-up |
| R-CR099-04 | MEDIUM | n/a | 本轮由 host-orchestrator 执行 WSL collector，不采用 user manual evidence fallback | N/A |

## 6. 阶段决策

- 结论：`PASS`
- 阻断项：无
- 需回修项：无
- 可进入：CP8 delivery readiness / risk acceptance
- 不授权项：Windows `.env`、账户原文、NAS、交易写、simulation/live、provider/lake/publish、非空持仓 / 交易日复测
