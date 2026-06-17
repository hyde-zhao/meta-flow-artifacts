---
status: "ready"
change_id: "CR-071"
release_decision: "READY_WITH_RISK"
created_at: "2026-06-16T09:13:26+08:00"
---

# CR071 Release Notes

## 摘要

CR071 已完成 MIG-A copy-first shadow root 迁移：`/home/hyde/workspace/local_backtest` 已复制到同级 `/home/hyde/workspace/quant-lab`，旧目录保留，不删除、不重命名。

本轮迁移明确排除 `reports/`、`data/`、`.venv/`、`node_modules/` 和 `.env`。CR071 关闭只确认本机 shadow root 已创建并通过静态验证，不表示日常工作根已切换。

## 用户可见变化

| 项目 | 状态 | 说明 |
|---|---|---|
| 新 shadow root | ready | `/home/hyde/workspace/quant-lab` 已存在。 |
| 旧 root | retained | `/home/hyde/workspace/local_backtest` 保持可用。 |
| reports/data/env/runtime | excluded | 目标根不包含这些目录或文件。 |
| reports Git 跟踪风险 | mitigated | `reports/` 已从本地 Git 跟踪中移除，文件保留。 |
| NAS 归档 | completed by CR072 | 本地 `reports/`、`data/` 和过程证据已归档到 NAS 并校验。 |

## 未改变的事项

| 范围 | 结论 |
|---|---|
| 远端 Git | 未执行 push、tag、branch/default branch 变更、history rewrite 或 force push。 |
| 数据湖 | 未切换 `MARKET_DATA_LAKE_ROOT`，未执行 provider fetch、lake write 或 catalog publish。 |
| QMT / MiniQMT | 未连接 runtime，未查询账户，未下单，未执行 simulation/live。 |
| 凭据 | 未复制或读取 `.env`、token、密码、cookie、session、private key 或账户事实。 |
| CR046 | 继续保持 user-paused，不恢复。 |

## 已知风险

| 风险 ID | 状态 | 说明 |
|---|---|---|
| R-CR071-01 | mitigated-before-cp8 | CP7 时目标 `reports/**` tracked deletion 风险已通过停止 Git 跟踪 reports 缓解。 |
| R-CR071-02 | accepted-risk | 目标根不是完整 runtime/data/report mirror，需要后续授权重建或恢复。 |
| R-CR071-03 | accepted-risk | 日常工作根尚未切换，后续要单独做 cutover。 |
| R-CR071-04 | accepted-risk | 两个本地根并存，若双向编辑会产生分叉风险。 |

## 回滚方式

默认回滚方式是继续使用旧根 `/home/hyde/workspace/local_backtest`。删除或清理 `/home/hyde/workspace/quant-lab` 不在 CR071 当前授权内，若需要释放空间或退回单根状态，应另起 root retirement / cleanup 门禁。

