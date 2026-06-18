---
story_id: "CR091"
story_slug: "QMT-STRATEGY-RUNNER"
status: "ready-for-verification"
owner: "meta-dev"
created_at: "2026-06-18T14:33:13+08:00"
scope: "offline implementation slice only"
---

# CR091 QMT Strategy Runner Implementation

## 实现前置检查

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CP5 已批准 | PASS | `process/checkpoints/CP5-CR091-QMT-STRATEGY-RUNNER-READINESS.md` | 用户于 `2026-06-18T14:16:02+08:00` 同意离线 implementation slice。 |
| 设计证据可读 | PASS | `process/stories/CR091-QMT-STRATEGY-RUNNER-LLD.md` | LLD 状态为 `approved-for-offline-implementation`。 |
| HLD / Test Plan / Research Notes 可读 | PASS | `docs/qmt/CR091-QMT-STRATEGY-RUNNER-HLD.md`、`docs/qmt/CR091-QMT-STRATEGY-RUNNER-TEST-PLAN.md`、`docs/qmt/CR091-RUNNER-RESEARCH-NOTES.md` | 设计边界保持 clean-room、package-driven、multi-factor-first、broker-neutral。 |
| 文件所有权 | PASS | 本文“实现对象清单” | 仅修改 CR091 批准范围内离线模块、checker、fixture、tests 与过程证据；未修改 `AGENTS.md`。 |
| 禁止范围 | PASS | 自动测试与 checker 输出 | 未启动 QMT / MiniQMT / XtQuant / gateway / runner runtime；未访问 NAS；未读取 `.env` / 凭据 / 账户 / 资金 / 持仓 / 委托 / 成交 / 日志原文。 |

## 实现对象清单

| 对象 | 路径 | 类型 | 验证方式 |
|---|---|---|---|
| public exports | `trading/strategy_runner/__init__.py` | package API | pytest import / checker import |
| adapter 合同与注册 | `trading/strategy_runner/adapters.py` | code | `tests/test_cr091_strategy_runner_contracts.py` |
| 目标组合快照 | `trading/strategy_runner/target_portfolio.py` | code | target portfolio / order intent tests |
| package loader | `trading/strategy_runner/package_loader.py` | code | manifest / checksum tests + checker |
| immutable cache | `trading/strategy_runner/cache.py` | code | active pointer tests + checker |
| 只读 gateway wrapper | `trading/strategy_runner/readonly_gateway.py` | code | fake gateway allow / deny tests |
| evidence writer | `trading/strategy_runner/evidence.py` | code | redaction / forbidden counters tests |
| 离线 checker | `scripts/check_cr091_strategy_runner_package.py` | script | `uv run --python 3.11 python ... --json` |
| 合同测试 | `tests/test_cr091_strategy_runner_contracts.py` | tests | pytest |
| fixture | `tests/fixtures/cr091_strategy_runner/*` | fixtures | pytest + checker |

## 设计契约映射

| 设计契约 | 实现位置 | 验证 |
|---|---|---|
| package-driven / multi-factor-first / broker-neutral runner 合同 | `AdapterRegistry` 默认注册 `MultifactorAdmissionAdapter`、`LegacyStrategyResultAdapter`、`StrategyPackageAdapter` | `test_multifactor_adapter_outputs_target_portfolio_and_order_intents`、`test_strategy_package_adapter_dispatches_loaded_payload` |
| 统一输出 `TargetPortfolioSnapshot` 和现有 `OrderIntentDraftV1` | `target_portfolio.py`、`adapters.build_pass_result()` 复用 `engine.order_intent_draft.build_order_intent_draft` | target/order intent tests；断言 `qmt_allowed=false`、`not_authorization=true` |
| manifest / checksum fail closed | `package_loader.load_strategy_package()`、`verify_checksums()` | `test_package_loader_fail_closed_on_bad_checksum`、`test_package_loader_requires_payload_checksum` |
| immutable local cache / active pointer fail closed | `cache.resolve_active_package()` | `test_active_pointer_fail_closed_when_not_immutable` |
| package authorization flags 全 false | `StrategyPackageAdapter`、`StrategyPackage.to_adapter_payload()` | `test_strategy_package_adapter_requires_all_authorization_flags_false` |
| readonly gateway 只允许 health / capabilities / query_positions | `readonly_gateway.ReadonlyGatewayClient.call()` | `test_readonly_fake_gateway_allows_only_listed_endpoints` |
| 默认 fake transport，不启动 runtime | `FakeReadonlyQmtTransport` | pytest + checker；未调用真实 transport / runtime |
| evidence 只输出 redacted summary 和 forbidden counters | `evidence.build_evidence_summary()`、`assert_redacted()` | `test_evidence_redacts_sensitive_material_and_counts_forbidden_operations`、`test_evidence_blocks_readonly_forbidden_counter_or_blocked_status`、`test_evidence_redaction_blocks_sensitive_payload` |
| 离线 checker 不读 `.env`、不访问 NAS、不启动 runtime | `scripts/check_cr091_strategy_runner_package.py` | checker 命令 PASS，输出 forbidden counters 全 0 |

## 单元测试与 Fixture 计划

| Fixture / 测试 | 覆盖点 | 状态 |
|---|---|---|
| `cr091_multifactor_admission_package_pass.json` | 多因子 PASS / WATCH 候选、risk/cost refs、operation counts 全 0 | PASS |
| `cr091_legacy_strategy_result_momentum.json` | legacy `StrategyResult` 等权目标组合与空目标 fail closed | PASS |
| `cr091_strategy_package/manifest.yaml` | manifest flags false、payload checksum、package adapter dispatch | PASS |
| `cr091_strategy_package/cache/active.json` | active pointer 只读解析、immutable marker | PASS |
| fake readonly gateway | health / capabilities / query_positions allow，交易写 endpoint deny | PASS |
| evidence tests | forbidden counters、脱敏 assurance、敏感 payload 阻断 | PASS |

## 最小实现切片

| Slice | 输出 | 局部验证 | 回滚点 |
|---|---|---|---|
| S1 数据合同与 adapter | `target_portfolio.py`、`adapters.py` | 多因子 / legacy / package adapter tests | 删除 `trading/strategy_runner` 新增模块 |
| S2 package/cache intake | `package_loader.py`、`cache.py`、package fixture | checksum / active pointer tests | 删除 package/cache 模块与 fixture |
| S3 readonly gateway / evidence | `readonly_gateway.py`、`evidence.py` | fake gateway / redaction tests | 删除 wrapper / evidence 模块 |
| S4 checker / CP6 证据 | checker、tests、本文、CP6 | checker + pytest + `git diff --check` | 删除 CR091 checker/tests/证据 |

## 平台差异处理

N/A。CR091 本切片不安装 Claude / Codex / OpenClaw 平台产物，不生成平台 agent / skill，不触发平台目录安装结构。

## 验证结果

| 命令 | 结果 | 说明 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_cr091_strategy_runner_contracts.py` | PASS，`13 passed in 0.15s` | 覆盖 adapter、package/cache、fake gateway、evidence。 |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_cr091_strategy_runner_package.py --package-root tests/fixtures/cr091_strategy_runner/cr091_strategy_package --json` | PASS，exit code 0 | 输出 `passed=true`，target_count=2，order_intent_count=2，forbidden counters 全 0。 |
| `git diff --check` | PASS | 无空白错误。 |

## Host Orchestrator 复核补丁

- 收紧 `StrategyPackageAdapter`：直接消费 package wrapper 时必须同时满足 `runtime_authorized`、`nas_operation_authorized`、`credential_read_authorized`、`account_query_authorized`、`trade_write_authorized` 全为 `false`。
- 收紧 manifest 校验：`payload_path` 必须存在且必须在 checksum 表中覆盖，否则 fail closed。
- 收紧 evidence 判定：readonly gateway 的状态、redaction status、raw payload 标志和 forbidden counters 会合并进最终 `pass` / `blocked` 结论。

## 未覆盖项

- 未运行全量测试套件：CR091 批准范围为离线切片，优先运行授权的单文件 contract tests；全量套件可能触及无关 provider / runtime / 大范围历史测试。
- 未执行真实 QMT / MiniQMT / XtQuant / gateway / runner runtime smoke：未授权，且 CP5 明确仍需后续逐 run runtime authorization。
- 未访问 NAS / `.env` / 凭据 / 账户 / 资金 / 持仓 / 委托 / 成交 / 日志原文：明确禁止范围。

## 设计缺口反馈

- 当前 `StrategyPackageAdapter` 仅作为 future package payload wrapper 分发到多因子 / legacy adapter；后续若引入真实策略包审批流，需要 CR093 或等价 package distribution CR 定义 NAS / cache promotion / active pointer governance。
- 下单 / 撤单 / simulation / live 仍必须拆到 CR092 或后续独立高风险 CR，不能复用本切片 checker 作为交易授权。

## 后续交接

- meta-qa 可直接运行本文“验证结果”中的两个离线命令。
- 重点复核 `qmt_allowed=false`、`not_authorization=true`、forbidden operation counters 全 0、fake gateway scope deny 和 evidence redaction。
- 不应在 CP7 中启动 QMT / MiniQMT / XtQuant / gateway / runner runtime，也不应访问 NAS、`.env`、凭据、账户、资金、持仓、委托、成交或日志原文。
