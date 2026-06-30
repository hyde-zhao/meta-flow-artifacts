# CR140 Phase 1 Contract Helpers Migration

## Scope

- 迁移 `experiments/lake_input_contract.py` 到 `engine/experiment_lake_input_contract.py`。
- 迁移 `experiments/training_snapshot_contract.py` 到 `engine/training_snapshot_contract.py`。
- 更新直接导入点：
  - `experiments/run_experiment_15_factor_framework.py`
  - `experiments/run_experiment_23_29_ml_factor_suite.py`
  - `tests/test_cr139_ml_lake_onboard_no_bypass.py`
  - `tests/test_cr139_training_snapshot_cutoff.py`

## Responsibility Decision

两个文件均为 CR139 contract / fixture validation helper，不是实验入口：

- `experiment_lake_input_contract` 负责实验读取 PIT lake 输入的显式契约，也支持测试 fixture 注入。
- `training_snapshot_contract` 负责训练快照 cutoff 的静态 / fixture 校验。

因此本轮纳入 Phase 1 归位，迁入 `engine/`，避免 Phase 1 后 `experiments/` 仍残留非实验 helper。

## Boundary

- 未访问真实数据湖。
- 未访问 NAS。
- 未触发 provider fetch。
- 未写 lake / catalog pointer。
- 未触发 QMT / MiniQMT / xtquant / gateway / simulation / live / trading runtime。

## Commands

| 命令 | 结果 | 说明 |
|---|---:|---|
| `git mv experiments/lake_input_contract.py engine/experiment_lake_input_contract.py` | PASS | 保留文件历史。 |
| `git mv experiments/training_snapshot_contract.py engine/training_snapshot_contract.py` | PASS | 保留文件历史。 |
| `uv run --python 3.11 pytest -q -p no:cacheprovider tests/test_cr139_ml_lake_onboard_no_bypass.py tests/test_cr139_training_snapshot_cutoff.py` | `8 passed` | CR139 lake input / training snapshot contract 行为保持。 |
| `uv run --python 3.11 python -m py_compile engine/experiment_lake_input_contract.py engine/training_snapshot_contract.py experiments/run_experiment_15_factor_framework.py experiments/run_experiment_23_29_ml_factor_suite.py tests/test_cr139_ml_lake_onboard_no_bypass.py tests/test_cr139_training_snapshot_cutoff.py` | PASS | 无语法错误。 |
| `git diff --check -- engine/experiment_lake_input_contract.py engine/training_snapshot_contract.py experiments/run_experiment_15_factor_framework.py experiments/run_experiment_23_29_ml_factor_suite.py tests/test_cr139_ml_lake_onboard_no_bypass.py tests/test_cr139_training_snapshot_cutoff.py` | PASS | 无 whitespace error。 |
| `rg -n "experiments\.lake_input_contract\|experiments\.training_snapshot_contract\|experiments/lake_input_contract\.py\|experiments/training_snapshot_contract\.py" experiments engine tests scripts docs process/changes process/checks` | PASS | 当前源码与测试无旧 import / 旧路径引用；命中仅历史 process 文档和 CR 影响面描述。 |

## Baseline Comparison

新增失败：无。

保持失败：无。

修复的基线失败：未在本切片验证集中确认。

## Decision

contract helpers 已纳入 Phase 1 归位。Phase 1 后可以声明 `experiments/` 的非实验 helper 清理覆盖了本轮识别出的 5 个文件，而不是只覆盖原计划中的 3 个文件。
