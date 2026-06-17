---
status: "PASS_WITH_RISK"
version: "1.0"
change_id: "CR-071"
title: "CR071 Copy-first Shadow Root Verification Report"
created_by: "host-orchestrator"
created_at: "2026-06-16T07:33:48+08:00"
validation_mode: "local-filesystem-static"
---

# CR071 Copy-first Shadow Root Verification Report

## 1. 验证结论

CR071 copy-first shadow root 已执行完成。`/home/hyde/workspace/quant-lab` 已创建并从 `/home/hyde/workspace/local_backtest` 复制，旧 root 保留。用户指定排除项 `reports/`、`data/`、`.venv/`、`node_modules/` 和 `.env` 在 target 中均不存在。

结论为 `PASS_WITH_RISK`：target 包含 `.git`，但因为 `reports/` 被明确排除，而源仓库中存在少量 tracked `reports/**` 文件，target 的 Git status 会显示这些 tracked report 文件为 deleted。这是用户排除 `reports/` 的预期后果，不代表执行越界。

## 2. 验证对象清单

| 对象 | 路径 / 命令 | 结果 | 说明 |
|---|---|---|---|
| Source root | `/home/hyde/workspace/local_backtest` | PASS | 旧目录仍存在。 |
| Target root | `/home/hyde/workspace/quant-lab` | PASS | 已创建。 |
| Copy command | `rsync -a --exclude ...` | PASS | 实际传输约 94MB。 |
| Excluded paths | `reports/ data/ .venv/ node_modules/ .env` | PASS | target 中均不存在。 |
| Git metadata | `/home/hyde/workspace/quant-lab/.git` | PASS_WITH_RISK | `.git` 已复制，tracked `reports/**` 显示 deleted。 |
| Manifest | `docs/release/COPY-FIRST-SHADOW-ROOT-MANIFEST-CR071.yaml` | PASS | 已复制到 target。 |

## 3. 验证追踪矩阵

| 需求 / 决策 | 验证方式 | 结果 | 证据 |
|---|---|---|---|
| DQ-CP2-CR071-02 copy-first root | target exists + source exists | PASS | shell preflight |
| DQ-CP2-CR071-03 exclusion list | `test ! -e target/<path>` | PASS | CP7 check |
| DQ-CP3-CR071-02 copy `.git` | `test -d target/.git` | PASS_WITH_RISK | Git status readable |
| DQ-CP3-CR071-03 target non-empty guard | preflight target absent | PASS | target absent before mkdir |
| DQ-CP5-CR071-02 frozen command | rsync stats | PASS | CP6 check |
| DQ-CP5-CR071-04 verification scope | target/source/excludes/Git status | PASS_WITH_RISK | this report |
| DQ-CP5-CR071-06 no external actions | command inventory | PASS | no NAS / no remote Git / no runtime |

## 4. 实际执行统计

| 指标 | 值 |
|---|---:|
| Number of files | 7,926 |
| Number of created files | 7,925 |
| Regular files transferred | 7,415 |
| Total file size | 94,071,175 bytes |
| Total transferred file size | 94,071,175 bytes |
| Literal data | 94,071,175 bytes |
| Deleted files | 0 |
| Target size after copy | 109M |

## 5. 剩余风险

| 风险 ID | 等级 | 状态 | 说明 |
|---|---|---|---|
| R-CR071-01 | MEDIUM | accepted-risk | target 的 Git status 显示 tracked `reports/**` deleted，因为 `reports/` 被用户明确排除。 |
| R-CR071-02 | MEDIUM | accepted-risk | target 不包含 data/report/venv/node_modules/.env，不能直接代表完整运行环境。 |
| R-CR071-03 | LOW | open-follow-up | 后续是否切换日常工作根到 target 仍未授权。 |

## 6. 不授权项确认

以下事项在 CR071 执行与验证中均未执行：

- NAS mount / copy / delete / archive promote。
- 远端 Git add/set-url/push/tag、branch/default branch governance、history rewrite、force push、remote deletion。
- `.env` 或任何凭据读取、打印、迁移。
- `MARKET_DATA_LAKE_ROOT` 切换、provider fetch、lake write、catalog publish。
- QMT / MiniQMT runtime、账户查询、下单、撤单、simulation/live。
- 删除、重命名或移动旧 root。
- CR046 recovery。
