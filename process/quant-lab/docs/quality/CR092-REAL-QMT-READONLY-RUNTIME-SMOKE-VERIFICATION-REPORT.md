---
report_id: "CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-VERIFICATION"
change_id: "CR-092"
status: "PASS_WITH_RISK"
created_at: "2026-06-18T17:05:00+08:00"
created_by: "host-orchestrator"
validation_mode: "static-only / design-and-evidence-review-only"
runtime_authorized: false
nas_authorized: false
credential_read_authorized: false
real_account_read_authorized: false
---

# CR092 Real QMT Readonly Runtime Smoke Verification Report

## 1. 验证结论

| 字段 | 内容 |
|---|---|
| 阶段 | CP7 verification |
| 结论 | `PASS_WITH_RISK` |
| 阻断项 | 0 |
| 实现缺陷 | 0 |
| 验证模式 | 静态验证、模板自检、单元测试、边界审查 |
| 未证明事项 | 真实 QMT / MiniQMT / XtQuant / gateway / runner runtime 可用性 |
| 不授权项 | runtime、NAS、凭据、真实账户、submit/cancel、simulation/live、provider/lake/catalog/publish |

CR092 CP7 只证明 CP6 交付的 manual guide、模拟账户 evidence template、单文件静态 checker 和测试资产满足当前 CP5 授权范围。模板自检通过不等于真实 runtime smoke 成功。

## 2. 验证范围

| 对象 | 路径 | 验证方式 | 结论 |
|---|---|---|---|
| Manual Guide | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-MANUAL-GUIDE.md` | 文档审查 | PASS |
| Evidence Template | `docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-EVIDENCE-TEMPLATE.yaml` | checker 自检 | PASS |
| Evidence Checker | `scripts/check_cr092_simulated_evidence.py` | 单元测试 / py_compile / 静态导入审查 | PASS |
| Checker Tests | `tests/test_cr092_simulated_evidence_checker.py` | pytest | PASS |
| CP6 evidence | `process/stories/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-IMPLEMENTATION.md` | 契约映射审查 | PASS |
| CP5 gate | `process/checkpoints/CP5-CR092-READONLY-RUNTIME-SMOKE-READINESS.md` | 人工审批审查 | PASS |

## 3. 追踪矩阵

| 需求 / 决策 | 设计证据 | 实现证据 | 验证证据 | 结论 |
|---|---|---|---|---|
| DQ-CP5-CR092-01 批准 LLD | LLD / CP5 checkpoint | Manual guide / template / checker | 本报告 §2 / CP7 check | PASS |
| DQ-CP5-CR092-02 模拟账户 evidence 合同 | TEST-PLAN §evidence contract | Evidence template / checker | checker 自检 + negative tests | PASS |
| DQ-CP5-CR092-03 CP5 不授权真实运行 | CP5 不授权范围 | checker 无 runtime/network 导入 | import guard test / static review | PASS |
| DQ-CP5-CR092-04 NAS、order-write、ledger 独立分流 | CR092 CR / CP5 checkpoint | 不产生 NAS / order-write 资产 | 文档审查 | PASS |
| DQ-CP5-CR092-05 接受 runtime 未证明风险 | CP5 风险接受项 | CP6 剩余风险 | CP7 结论 `PASS_WITH_RISK` | PASS |
| DQ-CP5-CR092-06 只允许 guide/template/checker | CP5 checkpoint | CP6 实现对象清单 | 文件影响范围审查 | PASS |

## 4. 设计契约验证

| 契约 | 验证方式 | 结果 |
|---|---|---|
| checker 必须显式传入单个 `--evidence` 文件 | CLI 设计审查 / tests | PASS |
| checker 不自动扫描目录 | CLI 设计审查 | PASS |
| evidence 必须声明 `account_mode: simulated` | tests negative case | PASS |
| forbidden counters 必须为 0 | tests negative case | PASS |
| scope 只允许 `health`、`capabilities`、`query_positions_readonly` | tests negative case | PASS |
| 禁止凭据、token、真实账户、NAS、order-write markers | tests negative case / static review | PASS |
| checker 不导入 runtime / network 模块 | `test_cr092_checker_does_not_import_runtime_or_network_modules` | PASS |

## 5. 命令证据

| # | 命令 | 结果 |
|---|---|---|
| 1 | `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | PASS，`process_link_health: ok` |
| 2 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr092_simulated_evidence_checker.py` | PASS，`5 passed` |
| 3 | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr092_simulated_evidence.py --evidence docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-EVIDENCE-TEMPLATE.yaml --json` | PASS，`passed=true` |
| 4 | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python -m py_compile scripts/check_cr092_simulated_evidence.py tests/test_cr092_simulated_evidence_checker.py` | PASS |
| 5 | `git diff --check -- docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-MANUAL-GUIDE.md docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-EVIDENCE-TEMPLATE.yaml scripts/check_cr092_simulated_evidence.py tests/test_cr092_simulated_evidence_checker.py docs/quality/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-VERIFICATION-REPORT.md docs/quality/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-TEST-REPORT.md docs/quality/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-REVIEW.md docs/quality/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-FIXES.md process/stories/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-IMPLEMENTATION.md process/checks/CP6-CR092-READONLY-RUNTIME-SMOKE-CODING-DONE.md process/checks/CP7-CR092-READONLY-RUNTIME-SMOKE-VERIFICATION-DONE.md process/context/CP6-CR092-IMPLEMENTATION-CONTEXT.yaml process/context/CP7-CR092-VERIFICATION-CONTEXT.yaml process/STATE.md process/changes/CR-INDEX.yaml process/changes/CR-092-REAL-QMT-READONLY-RUNTIME-SMOKE-DESIGN-GATE-2026-06-18.md process/changes/CR-091-FOLLOW-UP-TRACKING-2026-06-18.md process/checkpoints/CP5-CR092-READONLY-RUNTIME-SMOKE-READINESS.md process/stories/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-LLD.md docs/qmt/CR092-REAL-QMT-READONLY-RUNTIME-SMOKE-TEST-PLAN.md process/context/CP5-CR092-READINESS-CONTEXT.yaml` | PASS，无输出 |
| 6 | `uv run --python 3.11 meta-flow check cr-tracking --project-root .` | NON-BLOCKING，exit 1；active formal CRs 为 `CR-092`，失败项为 CR025 嵌套 active_change 和 CR019 follow-up 历史状态旧账 |

## 6. Agent Dispatch Evidence

| 检查项 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 子 agent 调度 | PASS_WITH_RISK | inline fallback | 本轮未启动真实 `meta-qa` 子代理；由 host-orchestrator 按 verification-execution 规则执行静态验证。 |
| 调度风险 | ACCEPTED_FOR_CP7 | 本报告 / CP7 check | 由于验证范围仅为静态产物和 checker，不涉及 runtime；风险在 CP8 继续显式接受。 |
| 不伪造调度 | PASS | 无 agent_id/thread_id 伪造 | CP7 不写虚假的 subagent evidence。 |

## 7. 剩余风险

| 风险 ID | 风险 | 严重度 | 状态 | 后续处理 |
|---|---|---|---|---|
| R-CR092-CP7-001 | 静态 CP7 不证明真实 runtime 可用 | High | accepted-for-CP8 | 真实只读 smoke 必须另起逐 run runtime 授权 |
| R-CR092-CP7-002 | 本轮未读取用户实际模拟账户 evidence，只验证模板 | Medium | accepted-for-CP8 | 用户提供证据后使用 checker 单文件验证 |
| R-CR092-CP7-003 | 未使用独立 meta-qa 子代理 | Medium | accepted-for-CP8 | CP8 决策中说明 inline fallback 风险 |
| R-CR092-CP7-004 | CR019 / CR025 历史账本旧账仍存在 | Low | non-blocking | 保留给 CR091-FU-04 ledger hygiene |

## 8. 不授权项

本 CP7 未授权、未执行、也未证明以下事项：

- QMT / MiniQMT / XtQuant / gateway / runner startup、connection、install 或 runtime。
- NAS access / list / read / copy / pull / write / publish / delete。
- `.env`、凭据、真实账号、真实账户、真实资金 / 持仓 / 委托 / 成交、未指定日志原文。
- submit/cancel、buy/sell。
- simulation/live。
- provider fetch、lake write、catalog publish。
- CR089 自动启动。
- CR020 gateway route restore。

## 9. 阶段决策

- CP7 结论：`PASS_WITH_RISK`
- 下一步：进入 CP8 delivery readiness / risk acceptance。
- CP8 必须继续说明：CR092 当前交付是 readiness / evidence guardrail，不是运行授权，不是 runtime smoke 成功证据。
