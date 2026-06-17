---
status: "ready"
change_id: "CR-071"
release_decision: "READY_WITH_RISK"
created_at: "2026-06-16T09:13:26+08:00"
---

# CR071 Migration

## 迁移结论

CR071 已完成本机 copy-first shadow root 迁移。迁移动作创建了 `/home/hyde/workspace/quant-lab`，并保留 `/home/hyde/workspace/local_backtest`。

## 迁移边界

| 对象 | 迁移状态 | 说明 |
|---|---|---|
| 代码与过程文档 | copied | 已复制到 target，并在后续证据同步中补齐 CR071 证据。 |
| `.git` | copied | target 为独立本地 Git 工作树。 |
| `reports/` | excluded | 不在 target；本地保留，并已由 CR072 归档到 NAS。 |
| `data/` | excluded | 不在 target；本地保留，并已由 CR072 归档到 NAS。 |
| `.venv/` | excluded | target 需要后续 `uv sync` 才能重建运行环境。 |
| `node_modules/` | excluded | target 不包含 Node 缓存或依赖目录。 |
| `.env` | excluded | 不复制，不读取，不迁移凭据。 |
| data lake root | unchanged | 不切换 `MARKET_DATA_LAKE_ROOT`。 |
| QMT / MiniQMT runtime | unchanged | 不连接、不验证、不执行交易相关动作。 |

## 兼容性

| 维度 | 结论 |
|---|---|
| 旧工作根兼容性 | 保持兼容，旧根仍可继续使用。 |
| 目标根可运行性 | 未保证，因 `.venv`、`.env`、`data/`、`reports/` 被排除。 |
| Git 远端兼容性 | 未改变远端；本轮只有只读核对。 |
| 历史路径引用 | 不批量改写历史过程证据中的 `local_backtest` 引用。 |

## 后续迁移候选

| 候选项 | 状态 | 触发条件 |
|---|---|---|
| root cutover | candidate | 用户决定把日常 shell/IDE/脚本入口切到 `/home/hyde/workspace/quant-lab`。 |
| target env rebuild | candidate | 需要在 target 中运行测试或开发命令。 |
| data/reports restoration | candidate | 需要 target 具备本地数据/报告访问能力。 |
| old root retirement | candidate | 确认 target 可独立承担工作后再考虑。 |
| remote Git governance | candidate | 需要同步远端或调整分支/default branch。 |

