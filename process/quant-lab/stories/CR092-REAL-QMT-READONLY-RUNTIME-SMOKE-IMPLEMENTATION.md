---
story_id: "CR092-READONLY-RUNTIME-SMOKE-DESIGN"
title: "CR092 Readonly Runtime Smoke Readiness Implementation"
status: "ready-for-verification"
created_at: "2026-06-18T16:55:00+08:00"
created_by: "host-orchestrator"
source_cr: "CR-092"
cp5_checkpoint: "process/checkpoints/CP5-CR092-READONLY-RUNTIME-SMOKE-READINESS.md"
runtime_authorized: false
nas_authorized: false
credential_read_authorized: false
real_account_read_authorized: false
---

# CR092 Readonly Runtime Smoke Readiness Implementation

## 实现前置检查

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP5 approved | PASS | `process/checkpoints/CP5-CR092-READONLY-RUNTIME-SMOKE-READINESS.md` | 用户回复“同意”。 |
| LLD ready | PASS | `process/stories/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-LLD.md` | full-lld。 |
| TEST-PLAN ready | PASS | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-TEST-PLAN.md` | design-and-evidence-review-only。 |
| runtime authorization | N/A | CP5 不授权 | 未执行 runtime。 |

## 实现对象清单

| 对象 | 路径 | 类型 | 说明 |
|---|---|---|---|
| Manual Guide | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-MANUAL-GUIDE.md` | 文档 | 给用户准备模拟账户 evidence 和运行静态 checker 的说明。 |
| Evidence Template | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-EVIDENCE-TEMPLATE.yaml` | YAML 模板 | 模拟账户 evidence 字段模板。 |
| Evidence Checker | `scripts/check_cr092_simulated_evidence.py` | guardrail / checker | 只读取显式传入的单个 evidence 文件并检查边界。 |
| Tests | `tests/test_cr092_simulated_evidence_checker.py` | 单元测试 | 覆盖 pass、真实账户拒收、非零 forbidden counter、越权 scope、无 runtime/network import。 |

## 设计契约映射

| 设计契约 | 实现位置 | 验证方式 |
|---|---|---|
| 只允许模拟账户 evidence | checker `account_mode == simulated` | `test_cr092_evidence_template_passes` |
| 拒收凭据 / token / 真实账户 / NAS / order-write | checker forbidden marker 和 counter 检查 | `test_cr092_checker_rejects_real_account_marker` / `test_cr092_checker_rejects_nonzero_forbidden_counter` |
| scope 限定 health / capabilities / query_positions_readonly | checker `ALLOWED_SCOPE` | `test_cr092_checker_rejects_forbidden_scope` |
| 不导入 runtime / network 模块 | checker import guard | `test_cr092_checker_does_not_import_runtime_or_network_modules` |
| 不自动扫描目录 | CLI 必须显式传 `--evidence` 单文件 | Manual guide + checker interface |

## 验证结果

| 命令 | 结果 |
|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr092_simulated_evidence_checker.py` | PASS，5 passed |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr092_simulated_evidence.py --evidence docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-EVIDENCE-TEMPLATE.yaml --json` | PASS，`passed=true` |
| `git diff --check -- <CR092 files>` | PASS，无输出 |

## 剩余风险

- checker 只能验证用户明确提供的 evidence 文件，不证明真实 runtime 可用。
- 模板通过不等于 smoke 成功；真实 smoke 仍需逐 run 授权和 CP7 evidence review。
- CR019 / CR025 历史账本旧账仍由 CR091-FU-04 单独处理。

## 不授权项

- QMT / MiniQMT / XtQuant / gateway / runner startup、connection、install 或 runtime。
- NAS access / list / read / copy / pull / write / publish / delete。
- `.env`、凭据、真实账户、真实资金 / 持仓 / 委托 / 成交、未指定日志。
- submit/cancel、buy/sell、simulation/live。
- provider fetch、lake write、catalog publish。
