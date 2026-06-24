---
cr_id: "CR-138"
task_id: "CR138-GUARDRAIL-01"
title: "Process Artifact Hygiene Guardrail Update"
status: "passed"
created_at: "2026-06-24T15:40:00+08:00"
created_by: "host-orchestrator"
scope: "CR138 CP4 前置 guardrail task"
runtime_authorization: "not_requested"
---

# CR138 Process Artifact Hygiene Guardrail

## 目标

在不新起 CR 的前提下，将 `scripts/check_process_artifact_hygiene.py` 修复纳入 CR138 的 CP4 前置 guardrail task，避免后续 Runner / QMT Gateway Story planning 与开发门禁被 CR138 合法产物误判为 `unclassified`。

## 实现对象

| 对象 | 变更 |
|---|---|
| `scripts/check_process_artifact_hygiene.py` | 限定 process git status 只扫描当前 `process/quant-lab` 子树；读取 active CR；新增 `current_cr_asset`、`current_workflow_shared_asset`、`current_guardrail_asset` 分类；将 CR138 follow-up 收敛触碰的 CR091 / CR098 tracking 台账纳入受控 workflow shared asset。 |
| `tests/test_cr132_process_artifact_hygiene.py` | 增加 CR138 active CR、功能命名 HLD/ADR、共享 workflow 文档、CR091 / CR098 follow-up tracking 台账和 guardrail source asset 分类测试。 |

## 边界

- 不新起 CR；本任务归入 CR138。
- 不修改 QMT Gateway runtime、MiniQMT、XtQuant、交易、账户、行情、订单、NAS、provider/lake/catalog 或 Git remote 逻辑。
- 不读取凭据、账户、环境变量或外部 runtime。
- 不放宽 unknown artifact 阻断；未知路径仍进入 `unclassified` 并阻断 runner development gate。

## 验证

| 命令 | 结果 |
|---|---|
| `uv run --python 3.11 pytest -q tests/test_cr132_process_artifact_hygiene.py` | PASS，14 passed |
| `uv run --python 3.11 python scripts/check_process_artifact_hygiene.py --json` | PASS，`unclassified=0`，`active_cr_numbers=["138"]` |
| `uv run --python 3.11 python -m py_compile scripts/check_process_artifact_hygiene.py tests/test_cr132_process_artifact_hygiene.py` | PASS |
| `git diff --check -- scripts/check_process_artifact_hygiene.py tests/test_cr132_process_artifact_hygiene.py` | PASS |
| `uv run --python 3.11 meta-flow cr check --project-root /home/hyde/workspace/quant-lab` | PASS |
| `uv run --python 3.11 meta-flow state check --project-root /home/hyde/workspace/quant-lab` | PASS |

## 2026-06-24 Follow-up 覆盖审计追加

CR138 follow-up CR 覆盖审计需要同步 `CR-091-FOLLOW-UP-TRACKING` 和 `CR-098-FOLLOW-UP-TRACKING`，否则 hygiene checker 会把这两个旧台账修改误判为 `unclassified`。本追加仍然不放宽 unknown artifact 阻断，只把本轮已知、受控、可追溯的 legacy follow-up tracking reconciliation 纳入共享 workflow 分类。

## 结论

CR138 的合法 CP2 / CP3 / CP4 前置产物已能被 hygiene checker 分类为当前 CR 资产或受控共享资产；sibling artifact repo 路径不再污染当前项目检查。该 guardrail 不改变 CR138 的 runtime 授权边界，后续仍需按 CP4 Story planning 推进。
