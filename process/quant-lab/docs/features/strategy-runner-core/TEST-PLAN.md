---
status: "active"
version: "1.0"
feature_id: "strategy-runner-core"
source_cr: "CR-128"
---

# Feature Test Plan: Strategy Runner Core

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-23 | host-orchestrator | 定义 CR128 offline runner core 的自动化和边界测试。 |

## 自动化范围

| Case ID | 覆盖内容 | 测试入口 |
|---|---|---|
| SRC-TC-01 | fixture package end-to-end 输出 pass `RunResult`。 | `tests/test_cr128_runner_core_mvp.py` |
| SRC-TC-02 | bad package / checksum / unknown adapter fail closed。 | `tests/test_cr128_runner_core_mvp.py` |
| SRC-TC-03 | `RunSpec` 禁止非 offline mode 和外部授权 flags。 | `tests/test_cr128_runner_core_mvp.py` |
| SRC-TC-04 | result 包含 target count、order intent count、evidence status 和 forbidden counters。 | `tests/test_cr128_runner_core_mvp.py` |
| SRC-TC-05 | public API / script entry 不读取 env、不访问 NAS、不启动 runtime。 | static / monkeypatch tests |
| SRC-TC-06 | CR091 / CR098 既有合同仍通过。 | `tests/test_cr091_strategy_runner_contracts.py`, `tests/test_cr098_runner_readonly_integration.py` |

## 禁止行为检查

| 禁止项 | 期望 |
|---|---|
| `.env` / credential read | 不出现读取入口；测试中 monkeypatch 后仍不触发。 |
| NAS access | 不调用 package exchange / NAS path。 |
| QMT / MiniQMT / XtQuant / gateway runtime | 不调用 `ReadonlyGatewayClient.from_transport` 或 runtime collector。 |
| provider / lake / catalog | 不导入或调用相关模块。 |
| submit/cancel/buy/sell/simulation/live | forbidden counters 必须为 0；order intent 仍为 draft。 |

## 验证命令

```bash
uv run --python 3.11 pytest -q tests/test_cr091_strategy_runner_contracts.py tests/test_cr098_runner_readonly_integration.py tests/test_cr128_runner_core_mvp.py
uv run --python 3.11 python scripts/check_cr091_strategy_runner_package.py --package-root tests/fixtures/cr091_strategy_runner/cr091_strategy_package --json
```
