---
status: "ready"
change_id: "CR-071"
release_decision: "READY_WITH_RISK"
created_at: "2026-06-16T09:13:26+08:00"
---

# CR071 Feedback

## 观察项

| ID | 观察信号 | 触发阈值 | 处理方式 |
|---|---|---|---|
| OBS-CR071-01 | 两个本地根同时被编辑 | 任一根出现未同步的业务代码或流程证据变更 | 暂停 cutover，先决定 authoritative root。 |
| OBS-CR071-02 | target 需要运行测试但缺少环境 | 用户或脚本需要在 `/home/hyde/workspace/quant-lab` 执行 `uv run` / pytest | 启动 target env rebuild gate。 |
| OBS-CR071-03 | target 需要访问 data/reports | 工作流依赖 `data/` 或 `reports/` | 启动 data/reports restoration 或 mount policy gate。 |
| OBS-CR071-04 | 需要同步远端 Git | 用户要求 push / branch / default branch 治理 | 启动 Git runtime authorization / execution manifest。 |

## 后续分流

| 项目 ID | 类型 | 状态 | 说明 |
|---|---|---|---|
| FU-CR071-01 | root cutover | candidate | 决定日常工作根是否切到 `/home/hyde/workspace/quant-lab`。 |
| FU-CR071-02 | old root policy | candidate | 决定 `/home/hyde/workspace/local_backtest` 长期保留、归档或删除策略。 |
| FU-CR071-03 | environment rebuild | candidate | 在 target 重建 `.venv` 并做最小 smoke。 |
| FU-CR071-04 | data/reports access | candidate | 设计 target 如何访问本地或 NAS 上的 data/reports。 |
| FU-CR071-05 | remote Git governance | candidate | 远端同步、分支治理或默认分支治理。 |
| FU-CR071-06 | CR046 recovery | deferred | 仍需用户明确恢复，CR071 不自动推进。 |

## 反馈入口

后续反馈应明确属于以下哪类：

| 类别 | 示例 | 默认路由 |
|---|---|---|
| copy result issue | target 缺文件、排除项错误、Git 状态异常 | CR071 follow-up / re-copy gate |
| root cutover | 希望切换 IDE/shell/脚本工作根 | 新 CR |
| data/runtime | 需要 data/reports/.env/.venv | 新 CR |
| remote Git | 需要 push、branch、default branch | 新 runtime authorization |
| trading/runtime | QMT、MiniQMT、账号、交易 | CR046 或独立 runtime gate |

