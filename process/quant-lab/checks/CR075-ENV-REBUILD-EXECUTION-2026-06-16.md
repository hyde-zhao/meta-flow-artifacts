---
check_id: CR075-ENV-REBUILD-EXECUTION-2026-06-16
cr_id: CR-075
status: PASS
created_at: "2026-06-16T23:55:28+08:00"
created_by: host-orchestrator
target_root: "/home/hyde/workspace/quant-lab"
legacy_root: "/home/hyde/workspace/local_backtest"
---

# CR075 Env Rebuild Execution

## Scope

CR075 CP2/CP3/CP5 已由用户回复“同意”批准。本执行证据只记录 target root 的基础环境重建，不包含 `.env`、reports/data、provider/lake/catalog、远端 Git、QMT/MiniQMT runtime、CR046 recovery、optional groups、full tests 或 destructive cleanup。

## Preflight

| 检查项 | 结果 | 证据 |
|---|---|---|
| CP2 人工门禁 | PASS | `process/checkpoints/CP2-CR075-ENV-REBUILD-BASELINE.md` |
| CP3 人工门禁 | PASS | `process/checkpoints/CP3-CR075-ENV-SMOKE-DESIGN-REVIEW.md` |
| CP5 人工门禁 | PASS | `process/checkpoints/CP5-CR075-ENV-SMOKE-READINESS.md` |
| CR tracking consistency | PASS | `python3 -B scripts/check_cr_tracking_consistency.py --project-root .` |
| 未授权项边界 | PASS | 本轮未读取 `.env`、reports/data，未执行 provider/lake/catalog、远端 Git、交易 runtime、CR046 recovery 或 destructive cleanup。 |

## Command Execution

| Step | Command | Exit Code | Result | Output Summary |
|---|---|---:|---|---|
| 1 | `uv sync --python 3.11` | 0 | PASS | 使用 CPython 3.11.15；创建 `.venv`；解析 149 个包；安装 47 个基础依赖。 |

## Environment Result

| 项 | 结果 |
|---|---|
| `.venv` | 已在 `/home/hyde/workspace/quant-lab/.venv` 创建 |
| Python | CPython 3.11.15 |
| 依赖范围 | 基础 `uv sync --python 3.11` |
| optional groups | 未执行 |
| cleanup | 未执行 |

## Decision

`PASS`。CR075 的 target 基础环境重建已完成，可进入最小 smoke 验证。
