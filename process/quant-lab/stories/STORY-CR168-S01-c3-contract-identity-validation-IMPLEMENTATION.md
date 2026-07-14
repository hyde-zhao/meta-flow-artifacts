# CR168-S01 Implementation — C3 合同、身份分域与输入校验

## 实现前置检查

CP5 Revision 2 已获批准；S01 无上游 Story 依赖，文件 owner 为本 Story。实现仅创建 repository-local `engine/economic_cost_evidence.py` 与 `tests/research/test_economic_cost_contracts.py`，未访问真实数据、凭据、环境变量、外部 ref、registry 或 runtime。

## 实现对象与设计契约映射

| 对象 | 文件 | 已实现合同 | 验证 |
|---|---|---|---|
| immutable raw/normalized/attachment/validation values | `engine/economic_cost_evidence.py` | S01 LLD §5/§6.1；family 1 attachment 与 families 2–9 semantic 分域 | valid nine-family typed result |
| N01..N10 validator | 同上 | S01 LLD §8.1；Domain Map §7 的 exact reason code 与稳定排序 | 10/10 negative matrix |
| subject-neutral semantic hash / tamper validator | 同上 | `quant-lab.economic-cost-input.v1` domain；identity 不进入 projection | 10 reruns→1；tamper BLOCKED |
| S01 contract suite | `tests/research/test_economic_cost_contracts.py` | EC-T01/05..10 的 S01 可实现部分；缺失静态分项假设不进入 S02 | 18 passed |

## 最小实现切片与结果

1. 定义只接收 `EconomicCostEvidenceInput` 的 normalizer；数值格式问题不抛出或修复，而是进入 N04。
2. 定义 `EconomicCostValidationResult(normalized_input, attachment_context, issues)`，作为 S02 唯一可消费的 typed 三元。
3. 校验 9 个字段族，并以 N01..N10 排序、`blocked` 优先的 availability 返回 fail-closed 结果。
4. 用 CR166 的 public canonical hash 原语产生不含 attachment identity 的 input semantic hash；外部声明 hash 不一致时给出 N10 BLOCKED。
5. 在 CP7 前置复核中补齐 S02 的输入前提：fee/tax/spread/slippage/impact 的每个静态参数缺失统一形成既有 N03 typed-unavailable，不允许 calculator 以 `None` 或隐式默认值继续。

S02 calculator/producer、S03 catalog、S04 Gate4 adapter、S05 fixture/QAC 均未实现，符合串行 Wave 所有权。

## 本地验证

| 命令 | 结论 | 结果 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python -m py_compile engine/economic_cost_evidence.py tests/research/test_economic_cost_contracts.py` | PASS | 静态编译通过。 |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_economic_cost_contracts.py` | PASS | 18 passed in 0.10s。 |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_walk_forward_oos_contracts.py tests/research/test_walk_forward_oos_qac.py` | PASS | 7 passed in 0.10s；合计 25 passed；确认 S01 新 import 未改变 CR166 C2 reserved-catalog/canonical contract。 |

初次回归命令误引用不存在的 `tests/research/test_strategy_evidence.py`，pytest 在收集前报告路径不存在、未运行任何测试；已通过 `rg --files tests` 更正为实际的 CR166 C2 contract/QAC 测试。此为命令路径纠正，不是源码测试失败。

## 边界与设计差异

无设计差异、无 design delta。模块不读写路径、不解引用 opaque refs、不运行外部操作；所有 C4、真实 TCA、capacity、runtime、registry 和 Git remote write 计数为 0。
