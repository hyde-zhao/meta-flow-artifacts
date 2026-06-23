---
status: "active"
version: "1.4"
feature_id: "strategy-runner-core"
source_cr: "CR-128"
---

# Feature Test Plan: Strategy Runner Core

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-23 | host-orchestrator | 定义 CR128 offline runner core 的自动化和边界测试。 |
| 1.1 | 2026-06-23 | host-orchestrator | 增加 CR133 RunSpec 文件入口和 CLI 测试范围。 |
| 1.2 | 2026-06-23 | host-orchestrator | 增加 CR134 runner evidence index 输出测试范围。 |
| 1.3 | 2026-06-23 | host-orchestrator | 增加 CR135 artifact bundle 写入、inspect/replay 和 blocked no-pass-bundle 测试范围。 |
| 1.4 | 2026-06-23 | host-orchestrator | 增加 CR136 bundle schema / compatibility validation、损坏 bundle fail-closed 和 CLI validate 测试范围。 |

## 自动化范围

| Case ID | 覆盖内容 | 测试入口 |
|---|---|---|
| SRC-TC-01 | fixture package end-to-end 输出 pass `RunResult`。 | `tests/test_cr128_runner_core_mvp.py` |
| SRC-TC-02 | bad package / checksum / unknown adapter fail closed。 | `tests/test_cr128_runner_core_mvp.py` |
| SRC-TC-03 | `RunSpec` 禁止非 offline mode 和外部授权 flags。 | `tests/test_cr128_runner_core_mvp.py` |
| SRC-TC-04 | result 包含 target count、order intent count、evidence status 和 forbidden counters。 | `tests/test_cr128_runner_core_mvp.py` |
| SRC-TC-05 | public API / script entry 不读取 env、不访问 NAS、不启动 runtime。 | static / monkeypatch tests |
| SRC-TC-06 | CR091 / CR098 既有合同仍通过。 | `tests/test_cr091_strategy_runner_contracts.py`, `tests/test_cr098_runner_readonly_integration.py` |
| SRC-TC-07 | JSON/YAML RunSpec 文件入口、spec-relative output 和 CLI JSON 输出。 | `tests/test_cr133_runner_spec_cli.py` |
| SRC-TC-08 | RunSpec 文件未知字段、外部授权 flag fail-closed。 | `tests/test_cr133_runner_spec_cli.py` |
| SRC-TC-09 | successful offline run 写入 lightweight evidence index。 | `tests/test_cr134_runner_evidence_index.py` |
| SRC-TC-10 | blocked run 不写误导性 pass index；CLI `--evidence-index-output` 行为一致。 | `tests/test_cr134_runner_evidence_index.py` |
| SRC-TC-11 | successful offline run 写入 artifact bundle，包含 result、evidence index、spec snapshot 和 manifest。 | `tests/test_cr135_runner_artifact_bundle.py` |
| SRC-TC-12 | bundle inspect/replay 只读取 bundle，不重新加载 strategy package；blocked run 不写 pass bundle；CLI `--bundle-output` / `--inspect-bundle` 行为一致。 | `tests/test_cr135_runner_artifact_bundle.py` |
| SRC-TC-13 | valid bundle 校验 manifest schema、required files、bytes/sha256、状态和授权边界一致性。 | `tests/test_cr136_runner_bundle_validation.py` |
| SRC-TC-14 | missing file、hash/size mismatch、schema mismatch、status mismatch、authorization mismatch 均 fail closed；CLI `--validate-bundle` 对坏 bundle 返回 blocked JSON。 | `tests/test_cr136_runner_bundle_validation.py` |

## 禁止行为检查

| 禁止项 | 期望 |
|---|---|
| `.env` / credential read | 不出现读取入口；测试中 monkeypatch 后仍不触发。 |
| NAS access | 不调用 package exchange / NAS path。 |
| QMT / MiniQMT / XtQuant / gateway runtime | 不调用 `ReadonlyGatewayClient.from_transport` 或 runtime collector。 |
| provider / lake / catalog | 不导入或调用相关模块。 |
| submit/cancel/buy/sell/simulation/live | forbidden counters 必须为 0；order intent 仍为 draft。 |
| corrupt / incompatible bundle | validate / inspect / replay 必须 fail closed，不得声明可 replay。 |

## 验证命令

```bash
uv run --python 3.11 pytest -q tests/test_cr091_strategy_runner_contracts.py tests/test_cr098_runner_readonly_integration.py tests/test_cr128_runner_core_mvp.py
uv run --python 3.11 pytest -q tests/test_cr133_runner_spec_cli.py
uv run --python 3.11 pytest -q tests/test_cr134_runner_evidence_index.py
uv run --python 3.11 pytest -q tests/test_cr135_runner_artifact_bundle.py
uv run --python 3.11 pytest -q tests/test_cr136_runner_bundle_validation.py
uv run --python 3.11 python scripts/check_cr091_strategy_runner_package.py --package-root tests/fixtures/cr091_strategy_runner/cr091_strategy_package --json
```
