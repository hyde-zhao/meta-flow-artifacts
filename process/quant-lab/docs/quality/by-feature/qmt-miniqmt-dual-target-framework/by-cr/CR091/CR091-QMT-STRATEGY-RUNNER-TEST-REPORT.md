---
change_id: "CR091"
title: "QMT Strategy Runner Offline CP7 Test Report"
owner: "meta-qa-critical"
created_at: "2026-06-18T14:59:24+08:00"
validation_mode: "mixed"
stage_decision: "PASS_WITH_RISK"
---

# CR091 QMT Strategy Runner Test Report

## 1. 测试结论

| 项目 | 内容 |
|---|---|
| 总结论 | `PASS_WITH_RISK` |
| 自动化测试 | PASS |
| 离线 checker | PASS |
| 静态边界扫描 | PASS |
| 语法检查 | PASS |
| CR tracking 可选检查 | NON-BLOCKING FAIL，历史旧账，不是 CR091 active 阻断 |
| 未覆盖 | 真实 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、账户、交易写、simulation/live、provider/lake/publish |

## 2. 测试环境和约束

| 项 | 内容 |
|---|---|
| project root | `/home/hyde/workspace/quant-lab` |
| Python 执行入口 | `uv run --python 3.11` |
| process route | `meta-flow workspace check` 通过，`routing_mode=symlink` |
| 测试策略来源 | `docs/qmt/CR091-QMT-STRATEGY-RUNNER-TEST-PLAN.md` |
| 验证环境 | 离线 fixture / fake transport；不需要 `VALIDATION-ENV.yaml` runtime approval |
| 禁止动作 | 未执行 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、凭据、账户、订单、simulation/live、provider/lake/publish |

## 3. 测试对象清单

| 对象 | 路径 | 测试类型 | 状态 |
|---|---|---|---|
| Adapter contract | `trading/strategy_runner/adapters.py` | contract unit | PASS |
| Target portfolio | `trading/strategy_runner/target_portfolio.py` | contract unit | PASS |
| Package loader | `trading/strategy_runner/package_loader.py` | package intake / fail closed | PASS |
| Cache pointer | `trading/strategy_runner/cache.py` | immutable cache / fail closed | PASS |
| Readonly gateway wrapper | `trading/strategy_runner/readonly_gateway.py` | fake transport allow/deny | PASS |
| Evidence summary | `trading/strategy_runner/evidence.py` | redaction / forbidden counters | PASS |
| Package checker | `scripts/check_cr091_strategy_runner_package.py` | CLI checker | PASS |
| Fixtures | `tests/fixtures/cr091_strategy_runner/` | fixture input | PASS |
| Test module | `tests/test_cr091_strategy_runner_contracts.py` | pytest | PASS |

## 4. 追踪矩阵

| 测试层级 | 覆盖目标 | 对应测试 / 命令 | 结果 |
|---|---|---|---|
| Static guardrail | 不出现 direct `xtquant` import、`.env`、requests/socket 真实入口 | `rg -n -S ...` static scan | PASS |
| Contract unit tests | 多因子、legacy、package adapter 统一合同 | `pytest -q tests/test_cr091_strategy_runner_contracts.py` | PASS |
| Package intake | manifest、checksum、payload checksum、authorization flags | pytest + checker | PASS |
| Multifactor fixture | PASS / WATCH 候选、risk/cost refs、operation counts | pytest + fixture read | PASS |
| Legacy fixture | momentum targets、score、empty target fail closed | pytest | PASS |
| Readonly fake gateway | health / capabilities / query_positions allow，submit_order deny | pytest | PASS |
| Redaction | token / secret / account / raw positions / qmt logs 不输出 | pytest + checker | PASS |
| Process routing | process symlink / artifact route health | `meta-flow workspace check` | PASS |

## 5. 命令结果

| # | 命令 | Exit | 关键结果 |
|---|---|---:|---|
| 1 | `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | 0 | `process_link_health: ok`，artifact git dirty `clean` |
| 2 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr091_strategy_runner_contracts.py` | 0 | `13 passed in 0.12s` |
| 3 | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr091_strategy_runner_package.py --package-root tests/fixtures/cr091_strategy_runner/cr091_strategy_package --json` | 0 | `passed=true`，`adapter_status=pass`，`readonly_reconciliation_status=ok` |
| 4 | `git diff --check` | 0 | 无输出 |
| 5 | static boundary scan | 0 | 未发现真实 runtime / sensitive read 入口；命中项均为 counter、fixture、本地 IO 或负向断言 |
| 6 | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python -m py_compile trading/strategy_runner/__init__.py trading/strategy_runner/adapters.py trading/strategy_runner/target_portfolio.py trading/strategy_runner/package_loader.py trading/strategy_runner/cache.py trading/strategy_runner/readonly_gateway.py trading/strategy_runner/evidence.py scripts/check_cr091_strategy_runner_package.py tests/test_cr091_strategy_runner_contracts.py` | 0 | 无输出 |
| 7 | `uv run --python 3.11 meta-flow check cr-tracking --project-root .` | 1 | 当前 active formal CRs: `CR-091`；失败项为历史 CR025 / CR019 follow-up 状态旧账 |

## 6. Checker JSON 摘要

| 字段 | 值 |
|---|---|
| `schema_version` | `cr091-strategy-runner-package-check-v1` |
| `passed` | `true` |
| `package_id` | `strategy-package-cr091-fixture-0.1.0` |
| `active_package_id` | `strategy-package-cr091-fixture-0.1.0` |
| `adapter_status` | `pass` |
| `target_count` | `2` |
| `order_intent_count` | `2` |
| `readonly_reconciliation_status` | `ok` |
| `not_authorization` | `true` |
| forbidden counters | 全部为 0 |

## 7. 覆盖缺口

| 缺口 | 原因 | 风险 | 下一步 |
|---|---|---|---|
| 未运行真实 QMT / MiniQMT / XtQuant / gateway / runner runtime | CP5 / 用户约束明确禁止 | HIGH | CP8 风险接受；真实验证需独立 runtime authorization |
| 未访问 NAS / `.env` / 凭据 / 账户 / 原始持仓 | 明确禁止 | HIGH | 保持不授权；如需验证需新 gate |
| 未运行全量测试套件 | CR091 本轮只授权离线切片定向测试 | MEDIUM | 后续更大范围回归需独立授权和范围评估 |
| CR tracking consistency 命令 exit 1 | 历史 CR025 / CR019 follow-up 旧账 | MEDIUM | 不阻断 CR091；host-orchestrator 后续治理 |

## 8. 8 维度验收

| # | 维度 | 状态 | 证据 | 说明 |
|---|---|---|---|---|
| 1 | 完整性 | PASS | CP6 object list + files present | 预期 code / script / tests / fixtures / implementation evidence 均存在 |
| 2 | 平台适配 | N/A | CR091 非 agent/skill/install 产物 | 不触发 Claude / Codex / OpenClaw 安装 |
| 3 | 验收标准覆盖 | PASS | HLD SC-CR091-01..06 trace | 每条成功标准均有实现与测试证据 |
| 4 | 安全合规 | PASS | static scan + checker counters | forbidden counters 全 0，无真实敏感入口 |
| 5 | 命名规范 | PASS | 路径审查 | Python package/script/test/fixture 命名符合项目约定 |
| 6 | Frontmatter 完整性 | N/A | code-project | CR091 不是 Agent / Skill frontmatter 产物 |
| 7 | 可安装性 | N/A | code-project | 无安装脚本或平台目录交付 |
| 8 | 文档覆盖 | PASS_WITH_RISK | HLD / LLD / TEST-PLAN / IMPLEMENTATION / CP6 / CP7 | 用户手册不在本轮输出范围，CP8 可决定是否补交付文档 |

## 9. 测试阶段决策

测试阶段建议 `PASS_WITH_RISK`。离线实现切片具备进入 CP8 的测试证据；CP8 必须明确风险接受：CR091 仍不证明真实 runtime ready，不授权交易主机、NAS、凭据、账户和交易写操作。
