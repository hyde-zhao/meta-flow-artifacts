# CR140 Phase 1.3 Data Lake Readiness Audit Migration

## Scope

- 迁移 `experiments/run_data_lake_readiness_audit.py` 到 `scripts/data_lake/run_data_lake_readiness_audit.py`。
- 更新 repo root 推导：`Path(__file__).resolve().parents[2]`。
- 更新 CLI `prog` 展示路径。
- 更新 `scripts/legacy/cr/cr012_limited_window_lake_repair.py` 的 repo root 推导与导入路径。
- 更新 `tests/test_data_lake_readiness_audit.py` 导入路径。
- 将新脚本加入 `scripts/quality/check_script_entrypoints.py` 的稳定入口清单。
- 更新 `docs/components/SCRIPT-ENTRYPOINTS.md` 稳定入口表。

## Responsibility Decision

该文件是 production_strict 数据湖 readiness audit 脚本入口，属于数据湖治理 / 审计操作面，不是研究实验入口。按职责归入 `scripts/data_lake/`。

## Boundary

- 未访问真实数据湖。
- 未访问 NAS。
- 未触发 provider fetch。
- 未写 lake / catalog pointer。
- 未触发 QMT / MiniQMT / xtquant / gateway / simulation / live / trading runtime。
- `--help` 验证只解析 CLI，不执行 audit、repair、fetch、apply 或写入。

## Commands

| 命令 | 结果 | 说明 |
|---|---:|---|
| `git mv experiments/run_data_lake_readiness_audit.py scripts/data_lake/run_data_lake_readiness_audit.py` | PASS | 保留文件历史。 |
| `uv run --python 3.11 pytest -q -p no:cacheprovider tests/test_data_lake_readiness_audit.py` | `11 passed` | readiness audit fixture 回归通过。 |
| `uv run --python 3.11 python -m py_compile scripts/data_lake/run_data_lake_readiness_audit.py scripts/legacy/cr/cr012_limited_window_lake_repair.py tests/test_data_lake_readiness_audit.py scripts/quality/check_script_entrypoints.py` | PASS | 无语法错误。 |
| `uv run --python 3.11 python scripts/data_lake/run_data_lake_readiness_audit.py --help` | PASS | 展示新稳定入口路径，未执行数据湖读取。 |
| `uv run --python 3.11 python scripts/legacy/cr/cr012_limited_window_lake_repair.py --help` | PASS | legacy repair 可导入新 readiness audit 路径，未执行 fetch/apply。 |
| `uv run --python 3.11 python scripts/quality/check_script_entrypoints.py` | FAIL | 失败项为 Phase 0 已登记的 26 个 `scripts/cr139_*` 根层入口；`missing_stable_entrypoints=[]`，本次新增稳定入口已覆盖。 |
| `git diff --check -- scripts/data_lake/run_data_lake_readiness_audit.py scripts/legacy/cr/cr012_limited_window_lake_repair.py tests/test_data_lake_readiness_audit.py scripts/quality/check_script_entrypoints.py docs/components/SCRIPT-ENTRYPOINTS.md` | PASS | 无 whitespace error。 |
| `rg -n "experiments\.run_data_lake_readiness_audit\|experiments/run_data_lake_readiness_audit\.py\|python experiments/run_data_lake_readiness_audit\.py" experiments engine tests scripts docs/components process/changes process/checks` | PASS | 当前源码、测试、脚本和组件文档无旧路径；命中仅历史 CR / process 记录和 CR140 影响面。 |

## Baseline Comparison

新增失败：无。

保持失败：

- `tests/test_script_entrypoint_naming.py::test_script_entrypoint_naming_guardrail_passes_current_repo` 对应的 26 个 CR139 root script naming debt 仍存在。

修复的基线失败：未在本切片验证集中确认；稳定入口缺失项已保持为空。

## Decision

Phase 1.3 可以继续进入 Phase 1 汇总验证。该切片改变脚本归属、入口路径和 import 面，不执行真实 lake audit 或 repair。
