# CR168-S02 Implementation — 确定性经济成本 Producer

## 实现前置检查

CP5 已批准，S01 已完成 CP7 `PASS_WITH_RISK` 并冻结 typed validation 三元。S02 文件所有权仅覆盖计算器、C3 evidence producer 与 producer 测试；未改动 CR-166 neutral envelope、canonical Gate 4、aggregate orchestration 或任何 C4/真实数据接口。

## 实现对象与设计契约映射

| 对象 | 文件 | 已实现合同 | 验证 |
|---|---|---|---|
| pure Decimal calculator | `engine/economic_cost_calculator.py` | S02 LLD §5/§8：五分项、precision=28、raw sum 后 final quantize、`net_return` | formula/proxy/rounding tests |
| unique public producer | `engine/economic_cost_evidence.py` | LLD §7：normalize→validate→issues short-circuit→calculate→produce | issue path calculator=0 |
| component identity/self validation | 同上 | feature DESIGN §5：subject-neutral component hash、fixture ref、v1 square_root only | 10 reruns / tamper test |
| S02 producer suite | `tests/research/test_economic_cost_producer.py` | EC-T01..T05/07..09 的 S02 可实现部分 | 12 passed |

## 最小实现切片与结果

1. 新建无 I/O 的 `economic_cost_calculator.py`，用 local Decimal context=28 计算 fee、tax、spread、slippage 和 static square-root impact；不使用 ADV、盘口、provider 或校准参数。
2. 在 S01 typed contract 上增加 immutable `CostBreakdownV1`、`EconomicCostEvidenceV1`、`EconomicCostBuildResult` 与 component self-validation。
3. 实现唯一 public `build_economic_cost_evidence`：issues 非空立即返回 blocked/typed_unavailable，保证 calculator invocation=0；仅 clean input 可计算并生成 present component。
4. 追加 exact basis、gross-return derivation、proxy 0/1/>1、zero reference、rounding、minor-unit、hash/tamper 覆盖。

## 本地验证

| 命令 | 结论 | 结果 |
|---|---|---|
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python -m py_compile engine/economic_cost_calculator.py engine/economic_cost_evidence.py tests/research/test_economic_cost_producer.py` | PASS | 静态编译通过。 |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_economic_cost_producer.py` | PASS | 12 passed in 0.04s。 |
| `PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/research/test_economic_cost_contracts.py tests/research/test_economic_cost_producer.py tests/research/test_walk_forward_oos_contracts.py tests/research/test_walk_forward_oos_qac.py` | PASS | 37 passed in 0.12s。 |
| static `rg` forbidden-operation / Gate4-private-helper scan + `git diff --check` | PASS | 无 file/network/process、canonical Gate4、aggregate 或 whitespace 违规命中。 |

## 边界与设计差异

无设计 delta。`economic_cost@v1` 仍是 fixture/static-only 的 C3 component：`no_real_tca_claim=true`、static limitations 被写入 component，且 `proxy>1` 只代表公式输入、从不产生 capacity claim。全仓 full-suite attribution、fixture/QAC/auth/CR155 回归和质量报告仍由 S05 拥有。
