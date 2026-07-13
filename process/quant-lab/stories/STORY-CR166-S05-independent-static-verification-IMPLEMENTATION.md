# CR166-S05 Implementation — Independent Static Verification Assets

## 实现前置检查

S01–S04 目标与相邻回归子集通过；S05 仅拥有 test/fixture 资产和质量证据。

## 实现对象与设计契约映射

| 对象 | 文件 | 设计契约 | 验证 |
|---|---|---|---|
| 12 QAC / hash tamper | `tests/research/test_walk_forward_oos_qac.py` | QAC-01..12 | 12/12 |
| authorization/static scan | `tests/research/test_walk_forward_oos_authorization.py` | RULE-166-15/16 | operation=0、forbidden import=0 |
| CR155 regression | `tests/research/test_walk_forward_oos_cr155_regression.py` | RULE-166-12 | promotion=0 |
| fixture families | `tests/fixtures/walk_forward_oos/` | SC-CR166-P01/P02/A01 | daily/ML=2，event N/A=1 |

## 最小切片与结果

新增 7 个 CR166 test modules 并登记 `tests/PROVENANCE.yaml`。目标与相邻回归 76 项通过；首次 full suite 1979 passed / 5 failed，5 项均归因于本 CR 的 taxonomy/design-surface/hygiene 登记缺口并已修复，随后进入全量复验。

## 边界与设计差异

无真实 fold/OOS、历史重算、外部系统或 runtime 验证。专题设计文件按既有 design-surface 规则迁入 `process/archive/design-cr-docs/`，属于治理路由修正，不改变批准设计。
