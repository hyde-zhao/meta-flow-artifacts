# CR091 QMT Strategy Runner LLD

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| v0.1 | 2026-06-18 | host-orchestrator | 定义 CR091 首版离线 runner 的文件边界、接口草案、测试映射和实施顺序。 |
| v0.2 | 2026-06-18 | host-orchestrator | 回填 CP5 人工批准结果，设计状态更新为可进入离线实现切片。 |

## 设计状态

状态：`approved-for-offline-implementation`

本 LLD 已于 `2026-06-18T14:16:02+08:00` 经用户回复“同意”通过 CP5。后续只允许离线实现切片、fake transport、fixture tests 和脱敏 evidence；仍不授权 QMT / MiniQMT / XtQuant / gateway / runner runtime、NAS、`.env` / 凭据 / 账户、submit / cancel、simulation / live、provider / lake / publish。

## 文件影响范围

| 类型 | 候选路径 | 动作 | 说明 |
|---|---|---|---|
| package | `trading/strategy_runner/__init__.py` | 新增 | runner public exports。 |
| package | `trading/strategy_runner/adapters.py` | 新增 | `StrategyAdapter` protocol 与 adapter registry。 |
| package | `trading/strategy_runner/target_portfolio.py` | 新增 | `TargetPortfolioSnapshot` 数据模型。 |
| package | `trading/strategy_runner/package_loader.py` | 新增 | manifest / checksum / fixture package 读取。 |
| package | `trading/strategy_runner/cache.py` | 新增 | 本地 immutable cache 和 active pointer 读取。 |
| package | `trading/strategy_runner/readonly_gateway.py` | 新增 | 只读 gateway client wrapper，默认 fake transport。 |
| package | `trading/strategy_runner/evidence.py` | 新增 | redacted evidence summary。 |
| script | `scripts/check_cr091_strategy_runner_package.py` | 新增 | 离线 package checker。 |
| tests | `tests/test_cr091_strategy_runner_contracts.py` | 新增 | adapter / package / evidence 合同测试。 |
| fixtures | `tests/fixtures/cr091_strategy_runner/` | 新增 | 手工构造脱敏 fixture。 |

## 接口草案

```python
class StrategyAdapter(Protocol):
    adapter_type: str
    input_schema: str

    def supports(self, payload: Mapping[str, object]) -> bool:
        ...

    def adapt(self, payload: Mapping[str, object], *, run_id: str) -> AdapterResult:
        ...
```

`AdapterResult` 必须包含：

- `status`
- `strategy_id`
- `target_portfolio`
- `order_intents`
- `blocked_reasons`
- `operation_counters`
- `not_authorization=true`

`TargetPortfolioSnapshot` 必须包含：

- `schema_version`
- `strategy_id`
- `source_run_id`
- `target_trade_date`
- `target_symbols`
- `target_weights`
- `score_refs`
- `risk_cost_refs`
- `lineage_refs`
- `limitations`
- `not_authorization=true`

## Adapter 映射

| Adapter | 输入 | 输出 | 阻断条件 |
|---|---|---|---|
| `MultifactorAdmissionAdapter` | `multifactor_strategy_admission_package_v1` | 目标组合 + 多条 `OrderIntentDraftV1` | schema 不匹配、operation counts 非 0、无 PASS / WATCH 候选、缺少 risk/cost refs。 |
| `LegacyStrategyResultAdapter` | `StrategyResult` dict | 等权目标组合 + order intent draft | 空 target、score 缺失、signal_date 缺失。 |
| `StrategyPackageAdapter` | future package payload | 目标组合 + order intent draft | 未声明 adapter_type、checksum 失败、manifest flags 非 false。 |

## 只读网关接口

首版只允许：

- `health`
- `capabilities`
- `query_positions`

默认实现必须使用 fake transport。真实 `StdlibQmtRestTransport` 只能在后续 runtime authorization 中由用户交易主机本地手工启用，且不能由本阶段自动化测试调用。

## Evidence 字段

| 字段 | 说明 |
|---|---|
| `schema_version` | `cr091-strategy-runner-evidence-v1` |
| `status` | `pass` / `blocked` |
| `run_id` | 本次离线 runner run id |
| `package_id` | package fixture 或 manifest id |
| `adapter_type` | 使用的 adapter |
| `strategy_id` | 策略 id |
| `target_count` | 目标数量 |
| `order_intent_count` | order intent draft 数量 |
| `readonly_reconciliation_status` | fake query_positions 对账状态 |
| `forbidden_operation_counters` | 禁止操作计数，全 0 才 pass |
| `redaction_assurance` | 原始账户 / 持仓 / secret / 日志输出均 false |

## 实施顺序

1. 新增数据模型和 adapter protocol。
2. 新增多因子 adapter，并用最小 fixture 覆盖 PASS / BLOCKED。
3. 新增 legacy strategy adapter，覆盖 momentum / empty target。
4. 新增 package loader 和 manifest checker。
5. 新增 fake readonly gateway wrapper。
6. 新增 evidence writer 和 redaction scan。
7. 新增 tests 和 checker 命令。
8. 运行 CP6 / CP7 离线验证。

## 回滚策略

- 若 adapter contract 不稳定，删除 `trading/strategy_runner/` 新增模块和 CR091 tests，保留 HLD / LLD 作为未实现方案。
- 若多因子 package 字段不足，保留 `LegacyStrategyResultAdapter`，把 `MultifactorAdmissionAdapter` 标记为 blocked，并回到 CR039/CR047 修正输入。
- 若只读 gateway wrapper 引入边界风险，首版移除 gateway wrapper，仅保留 package + adapter + evidence 离线验证。

## 不授权项

本 LLD 不授权 NAS、`.env`、凭据、账户、资金、持仓、委托、成交、日志原文、QMT / MiniQMT / XtQuant / gateway / runner runtime 启动、submit / cancel、simulation / live、provider / lake / publish。
