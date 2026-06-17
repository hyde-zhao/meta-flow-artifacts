---
status: "ready"
change_id: "CR-071"
release_decision: "READY_WITH_RISK"
created_at: "2026-06-16T09:13:26+08:00"
---

# CR071 Rollback

## 回滚目标

回滚到 CR071 前的工作方式：继续使用 `/home/hyde/workspace/local_backtest` 作为工作根。由于 CR071 采用 copy-first，旧根保留，默认无需恢复数据。

## 可执行回退路径

| 场景 | 处理方式 | 授权要求 |
|---|---|---|
| 不想使用新 root | 继续使用 `/home/hyde/workspace/local_backtest`。 | 无需额外授权。 |
| 目标 root 需要重建 | 暂停使用 `/home/hyde/workspace/quant-lab`，另起 cleanup/re-copy 门禁。 | 需要授权。 |
| 想删除目标 root | 另起 destructive cleanup 门禁并确认目标内容可丢弃。 | 需要明确删除授权。 |
| 需要恢复 reports/data 到目标 | 使用 CR072 NAS 归档或本地保留副本设计恢复方案。 | 需要 data/runtime restoration CR。 |
| 需要恢复 Git 远端事实 | N/A，本轮未写远端。 | 如需远端治理，另起 Git runtime authorization。 |

## 不可自动回滚项

| 项目 | 原因 |
|---|---|
| 删除 `/home/hyde/workspace/quant-lab` | 破坏性操作，未获授权。 |
| 删除 `/home/hyde/workspace/local_backtest` | 明确不授权，且旧根是当前回退路径。 |
| 复制 `.env` 或凭据 | 安全边界不允许。 |
| 远端 Git、data lake、runtime 状态 | 本轮未改变，无需回滚。 |

## 验证

回退验证只需确认：

| 检查 | 预期 |
|---|---|
| `test -d /home/hyde/workspace/local_backtest` | PASS |
| 在旧根继续运行本地命令 | 按原工作流执行 |
| 远端 master | 仍为 `3ade165e8b1edad031a911490cf6c1cee942616e`，除非后续另有授权 |

