---
report_id: "CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-TEST"
change_id: "CR-092"
status: "PASS"
created_at: "2026-06-18T17:05:00+08:00"
created_by: "host-orchestrator"
---

# CR092 Real QMT Readonly Runtime Smoke Test Report

## 测试摘要

| 字段 | 内容 |
|---|---|
| 测试范围 | 模拟账户 evidence template、静态 checker、checker 单元测试 |
| 测试模式 | 本地静态 / fixture-only |
| 结论 | `PASS` |
| 测试用例数 | 5 |
| 真实 runtime | 未授权、未执行 |

## 测试用例

| 用例 | 目标 | 结果 |
|---|---|---|
| `test_cr092_evidence_template_passes` | 模板满足 checker 合同 | PASS |
| `test_cr092_checker_rejects_real_account_marker` | 拒收真实账户 marker | PASS |
| `test_cr092_checker_rejects_nonzero_forbidden_counter` | 拒收非零禁止动作计数 | PASS |
| `test_cr092_checker_rejects_forbidden_scope` | 拒收越权 scope | PASS |
| `test_cr092_checker_does_not_import_runtime_or_network_modules` | checker 不导入 runtime / network 模块 | PASS |

## 命令结果

| 命令 | 结果 |
|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr092_simulated_evidence_checker.py` | PASS，`5 passed` |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr092_simulated_evidence.py --evidence docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-EVIDENCE-TEMPLATE.yaml --json` | PASS，`passed=true` |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python -m py_compile scripts/check_cr092_simulated_evidence.py tests/test_cr092_simulated_evidence_checker.py` | PASS |

## 覆盖说明

| 维度 | 覆盖结论 | 说明 |
|---|---|---|
| 正向路径 | PASS | 模板自检通过。 |
| 负向路径 | PASS | 覆盖真实账户 marker、非零 forbidden counter、越权 scope。 |
| 安全边界 | PASS | checker 无 runtime / network 导入；不读取目录或隐式路径。 |
| 集成验证 | PASS_WITH_RISK | 仅验证模板和 checker，不接触真实 QMT runtime。 |
| 回归范围 | PASS_WITH_RISK | 仅运行 CR092 定向测试，未运行全量套件。 |

## 不授权项

本测试报告不代表授权或执行 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、真实账户、submit/cancel、simulation/live、provider/lake/catalog/publish。
