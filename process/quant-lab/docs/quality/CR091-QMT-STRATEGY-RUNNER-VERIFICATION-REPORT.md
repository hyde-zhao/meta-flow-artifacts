---
change_id: "CR091"
title: "QMT Strategy Runner Offline CP7 Verification"
owner: "meta-qa-critical"
created_at: "2026-06-18T14:59:24+08:00"
validation_mode: "mixed"
sut_type: "code-project"
stage_decision: "PASS_WITH_RISK"
route: "host-orchestrator"
---

# Verification: CR091 QMT Strategy Runner

## 1. 结论

| 项目 | 内容 |
|---|---|
| 阶段决策 | `PASS_WITH_RISK` |
| validation_mode | `mixed`，但仅包含离线 static / contract test / fixture package checker / review；不包含真实 runtime |
| 路由 | `host-orchestrator`，进入 CP8 风险接受输入 |
| workflow eval | N/A，CR091 本轮为 code-project 离线实现切片，不是 generated workflow / prompt-skill workflow |
| runtime 结论 | 未验证。未启动 QMT / MiniQMT / XtQuant / gateway / runner runtime |

离线 CP7 验证范围内的必跑命令均通过：workspace route check 通过，CR091 contract tests `13 passed`，offline package checker `passed=true`，`git diff --check` 无输出，CR091 新增 Python 文件 `py_compile` 无输出。静态边界扫描未发现 CR091 文件直接 import `xtquant`、读取 `.env` / `os.environ` / `dotenv`、使用 `requests` / `socket` / `subprocess` 真实入口，命中项为 forbidden counter 字段、阻断测试、fixture 本地读取和本地 evidence 写出函数。

结论为 `PASS_WITH_RISK` 的原因是：本轮只证明 runner 合同、adapter、package/cache intake、fake readonly gateway wrapper、redacted evidence 和 forbidden counters 的离线封闭性；仍不证明真实 QMT / MiniQMT / XtQuant / gateway / runner runtime 可用，不证明策略可交易。该风险不阻断离线 CP7，但必须汇入 CP8 风险接受。

## 2. 验证范围

| 分类 | 范围 | 结论 |
|---|---|---|
| In scope | `trading/strategy_runner/` | 已验证 adapter、target portfolio、package loader、cache、readonly gateway fake transport、evidence |
| In scope | `scripts/check_cr091_strategy_runner_package.py` | 已验证本地 fixture package checker JSON 输出 |
| In scope | `tests/test_cr091_strategy_runner_contracts.py` | 已复跑并通过 |
| In scope | `tests/fixtures/cr091_strategy_runner/` | 已读取 fixture 结构并通过 checker / tests |
| In scope | CP5 / CP6 / Implementation evidence | 已回链 CP5 批准、CP6 PASS、实现对象和设计契约 |
| Out of scope | QMT / MiniQMT / XtQuant / gateway / runner runtime | 明确未授权且未执行 |
| Out of scope | NAS、`.env`、凭据、账户、资金、持仓、委托、成交、日志原文 | 明确未授权且未读取 |
| Out of scope | submit / cancel、buy / sell、simulation / live、provider / lake / publish | 明确未授权且未执行 |
| Out of scope | 全量测试套件 | CR091 CP7 仅授权单文件离线 contract tests；全量套件可能触及无关历史范围 |

`docs/quality/TEST-STRATEGY.md` 未在本轮写入，原因是用户指定了严格输出范围。等价测试策略来源为 `docs/qmt/CR091-QMT-STRATEGY-RUNNER-TEST-PLAN.md`，其测试层级、fixture、禁止操作断言和允许命令已被本报告消费。

## 3. 验证对象清单

| 对象 | 路径 | 验证方式 | 结果 |
|---|---|---|---|
| Adapter registry and adapters | `trading/strategy_runner/adapters.py` | pytest + code review + static scan | PASS |
| Target portfolio contract | `trading/strategy_runner/target_portfolio.py` | pytest + code review | PASS |
| Package manifest / checksum loader | `trading/strategy_runner/package_loader.py` | pytest + checker + path escape review | PASS |
| Immutable cache / active pointer | `trading/strategy_runner/cache.py` | pytest + checker + path escape review | PASS |
| Readonly fake gateway wrapper | `trading/strategy_runner/readonly_gateway.py` | pytest + static scan | PASS |
| Evidence summary / redaction | `trading/strategy_runner/evidence.py` | pytest + checker + sensitive pattern review | PASS |
| Public exports | `trading/strategy_runner/__init__.py` | import via tests + py_compile | PASS |
| Offline checker | `scripts/check_cr091_strategy_runner_package.py` | direct JSON command + static scan | PASS |
| Contract tests | `tests/test_cr091_strategy_runner_contracts.py` | pytest | PASS |
| Fixtures | `tests/fixtures/cr091_strategy_runner/` | pytest + checker + scan | PASS |
| Process state summary | `process/STATE.md` CR091 `cr_tracking` / `agent_lifecycle` 摘要 | targeted read | PASS_WITH_RISK，active CR 为 CR091；历史 CR tracking 旧账存在 |

## 4. 验证追踪矩阵

| 来源 | 设计 / 验收点 | 实现证据 | 验证证据 | 结果 |
|---|---|---|---|---|
| HLD SC-CR091-01 | 多因子 admission package 为一等输入 | `MultifactorAdmissionAdapter`、fixture admission JSON | `test_multifactor_adapter_outputs_target_portfolio_and_order_intents` | PASS |
| HLD SC-CR091-02 | 至少 3 类 adapter | `MultifactorAdmissionAdapter`、`LegacyStrategyResultAdapter`、`StrategyPackageAdapter` | pytest adapter / package tests | PASS |
| HLD SC-CR091-03 | 输出统一到 `TargetPortfolioSnapshot` + `OrderIntentDraftV1` | `target_portfolio.py`、`build_pass_result()` | pytest 断言 target/order intent、`qmt_allowed=false`、`not_authorization=true` | PASS |
| HLD SC-CR091-04 | 离线验证封闭，forbidden counters 全 0 | `zero_cr091_operation_counters()`、manifest / fixtures / evidence | checker JSON counters 全 0；static scan | PASS |
| HLD SC-CR091-05 | 只读 gateway 只允许 health / capabilities / query_positions | `ReadonlyGatewayClient.call()` | `test_readonly_fake_gateway_allows_only_listed_endpoints` | PASS |
| HLD SC-CR091-06 | evidence 可脱敏审计 | `build_evidence_summary()`、`assert_redacted()` | evidence tests + checker JSON | PASS |
| LLD 文件影响范围 | 仅新增 runner package、checker、tests、fixtures | CP6 implementation object list | file read + py_compile + tests | PASS |
| LLD 不授权项 | 不授权 NAS/runtime/credentials/trading writes | implementation evidence + scan | static boundary scan + commands executed | PASS |
| CP5 DQ-CP5-CR091-06 | 自动验证只允许 fake transport | `FakeReadonlyQmtTransport` | pytest + checker | PASS |
| CP5 DQ-CP5-CR091-08 | 首版不证明真实可交易 | 本报告风险 R-CR091-CP7-001 | CP8 risk acceptance input | PASS_WITH_RISK |

## 5. 设计契约验证清单

| 契约 | 来源 | 验证方式 | 阻断性 | 结果 |
|---|---|---|---|---|
| Runner 不直接 import `xtquant` | HLD §4、Test Plan Static guardrail | CR091 范围静态扫描 | BLOCKING | PASS |
| 不读取 `.env`、凭据、账号 | HLD §4 / §8、CP5 不授权项 | static scan + code review | BLOCKING | PASS |
| Package schema / checksum fail closed | HLD §8、LLD Adapter 映射 | pytest bad checksum / missing checksum | BLOCKING | PASS |
| Manifest flags 必须全 false | CP5 DQ-CP5-CR091-05/06/07 | pytest `account_query_authorized=True` 被阻断 | BLOCKING | PASS |
| Active pointer 只能指向 immutable local cache | LLD 文件影响范围 / 回滚策略 | pytest missing `.immutable` 被阻断 | BLOCKING | PASS |
| Fake readonly gateway deny 写 endpoint | HLD SC-CR091-05 | pytest `submit_order` -> `blocked_scope_denied` | BLOCKING | PASS |
| Evidence 合并 readonly counter 并 fail closed | Implementation host review patch | pytest `gateway_socket_open=1` -> blocked | BLOCKING | PASS |
| Evidence 阻断敏感 payload | HLD SC-CR091-06 | pytest `assert_redacted({"token": ...})` raises | BLOCKING | PASS |
| `qmt_allowed=false` / `not_authorization=true` | HLD risk mitigation、LLD interface | pytest assertions | BLOCKING | PASS |
| 真实 runtime 后续逐 run 授权 | CP5 DQ-CP5-CR091-06 | 本轮未执行 runtime command | BLOCKING for runtime, N/A for offline CP7 | PASS_WITH_RISK |

## 6. 分层验证计划

| 层级 | 验证动作 | 命令 / 方法 | 结果 |
|---|---|---|---|
| Workspace route | process symlink 与 artifact route health | `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | PASS |
| Contract unit tests | adapter、package/cache、fake gateway、evidence | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr091_strategy_runner_contracts.py` | PASS，13 passed |
| Fixture package checker | manifest/checksum/cache/adapter/fake readonly/evidence JSON | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr091_strategy_runner_package.py --package-root tests/fixtures/cr091_strategy_runner/cr091_strategy_package --json` | PASS，`passed=true` |
| Static whitespace | Git whitespace check | `git diff --check` | PASS，无输出 |
| Static boundary scan | `xtquant`、`.env`、`os.environ`、`dotenv`、`requests`、`socket`、forbidden action tokens | `rg -n -S ... trading/strategy_runner scripts/check_cr091... tests/... fixtures/...` | PASS，未发现真实运行入口 |
| Syntax | CR091 新增 Python 文件 py_compile | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python -m py_compile ...` | PASS，无输出 |
| CR tracking optional | 活跃 CR 与历史旧账识别 | `uv run --python 3.11 meta-flow check cr-tracking --project-root .` | NON-BLOCKING FAIL，当前 active formal CR 为 CR091，报错为历史 CR025 / CR019 follow-up 旧账 |
| Runtime integration | QMT / gateway / NAS / credentials / account / submit / live | 未授权，不执行 | N/A，风险进入 CP8 |

## 7. 自动化验证结果

| # | 命令 | Exit | 关键输出 | 结论 |
|---|---|---:|---|---|
| 1 | `uv run --python 3.11 meta-flow workspace check --project-root /home/hyde/workspace/quant-lab` | 0 | `process_link_health: ok`，routing mode `symlink` | PASS |
| 2 | `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr091_strategy_runner_contracts.py` | 0 | `13 passed in 0.12s` | PASS |
| 3 | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr091_strategy_runner_package.py --package-root tests/fixtures/cr091_strategy_runner/cr091_strategy_package --json` | 0 | `passed=true`，`target_count=2`，`order_intent_count=2`，forbidden counters 全 0 | PASS |
| 4 | `git diff --check` | 0 | 无输出 | PASS |
| 5 | static boundary scan | 0 | 仅命中 counter、fixture、本地 file IO、阻断测试和 fake gateway 注释 | PASS |
| 6 | `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python -m py_compile ...` | 0 | 无输出 | PASS |
| 7 | `uv run --python 3.11 meta-flow check cr-tracking --project-root .` | 1 | summary: active formal CRs `CR-091`; errors 指向历史 `CR-025` nested active_change 与 CR019 follow-up 旧状态 | PASS_WITH_RISK，非 CR091 实现阻断 |

## 8. Prompt / Skill Fixture 验证

N/A。CR091 本轮不是 Prompt / Skill / generated workflow 产物。等价 fixture 验证为本地 JSON / YAML package fixtures 与 contract tests，已覆盖多因子、legacy、package、fake query_positions 和 evidence redaction。

## 9. 平台适配验证

N/A。CR091 本轮不交付 Claude / Codex / OpenClaw agent、skill 或安装脚本，不触发平台安装路径。代码运行环境验证使用 `uv run --python 3.11`，符合项目 Python 依赖管理约定。

## 10. 人工 / 语义质量审查

| 审查项 | 结果 | 说明 |
|---|---|---|
| 需求一致性 | PASS | 实现对象与 CP5 批准的 `trading/strategy_runner` 薄模块 + checker + fixtures/tests 一致 |
| 场景覆盖 | PASS | 覆盖 multi-factor first、legacy adapter、future package adapter、checksum/cache、fake gateway、redaction |
| 禁止边界 | PASS | 未运行真实 runtime、未访问 NAS、未读 `.env` / 凭据 / 账户、未执行交易写 |
| Fail closed | PASS | schema、checksum、manifest flags、active pointer、gateway scope、redaction 均有负向测试 |
| Happy path 偏差 | PASS_WITH_RISK | happy path 仅为离线 fixture，不代表真实交易主机 |
| 文档可用性 | PASS | HLD / LLD / TEST-PLAN / IMPLEMENTATION / CP6 均可回链；未写全局 TEST-STRATEGY 因用户限定输出范围 |

## 11. 问题清单

| ID | 等级 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| FINDING-CR091-CP7-001 | N/A | none-found | 本轮未发现需 meta-dev 回修的 BLOCKER / HIGH / MEDIUM / LOW 实现缺陷 | 无需回修 |
| OBS-CR091-CP7-001 | INFO | open-process-debt | `meta-flow check cr-tracking` exit 1，但 summary 显示 active formal CRs 为 `CR-091`，报错集中于历史 CR025 / CR019 follow-up | 不阻断 CR091 离线 CP7；由 host-orchestrator 在后续治理 |

## 12. 剩余风险

| 风险 ID | 等级 | 风险 | 当前处理 | CP8 输入 |
|---|---|---|---|---|
| R-CR091-CP7-001 | HIGH | 离线 CP7 不证明真实 QMT / MiniQMT / XtQuant / gateway / runner runtime 可用，也不证明策略可交易 | 不阻断离线 CP7；保持不授权真实 runtime | 必须作为 CP8 `risk_acceptance` 决策项 |
| R-CR091-CP7-002 | MEDIUM | CR tracking 检查存在历史旧账，可能干扰后续全局门禁输出 | 当前 active formal CR 为 CR091，本轮不修改历史跟踪文件 | 进入 follow-up tracking 或由 host-orchestrator 单独清理 |
| R-CR091-CP7-003 | LOW | `write_evidence_summary()` 支持本地写文件，但本轮 checker 未调用写出路径 | 函数写出前执行 `assert_redacted()`；CP7 未调用写出命令 | 若后续启用 evidence 落盘，需限定输出目录和审计路径 |

## 13. 阶段决策与 CP8 输入

阶段决策：`PASS_WITH_RISK`。

允许推进条件：

- CR091 离线 contract tests、checker、static scan、syntax check 均通过。
- 无需 meta-dev 回修。
- 不把本结论解释为真实运行授权。

CP8 必须展示和收集的风险接受项：

| 决策类型 | 推荐处理 | 备选方案 | 影响 / 风险 | 回退 / 切换条件 |
|---|---|---|---|---|
| `risk_acceptance` | 接受 CR091 离线 runner 合同通过，但标记真实 runtime 未验证 | 要求新建 runtime authorization gate 后再进入 delivered；或暂停 CR091 关闭 | 推荐方案可关闭离线 implementation slice；备选会延后交付但补真实环境证据 | 用户要求真实 QMT / MiniQMT / XtQuant / gateway / runner 验证时，创建独立 runtime gate |
| `follow_up_tracking` | 将 CR tracking 历史旧账列为非阻断流程债 | 本轮强制修复历史 CR 跟踪；或忽略 | 推荐方案避免扩大 CR091 范围；强制修复会越界进入历史治理 | 若 CP8 / 后续 CR 门禁被旧账阻断，再单独发起治理任务 |

不授权项继续有效：本 CP7 不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、`.env` / 凭据 / 账号 / 账户 / 资金 / 持仓 / 委托 / 成交 / 日志原文、submit / cancel、buy / sell、simulation / live、provider / lake / catalog / publish。
