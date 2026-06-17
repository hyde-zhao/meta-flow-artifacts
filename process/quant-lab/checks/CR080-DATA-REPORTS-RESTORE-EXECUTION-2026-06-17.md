---
check_id: "CR080-DATA-REPORTS-RESTORE-EXECUTION"
cr_id: "CR-080"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-17T07:05:05+08:00"
started_at: "2026-06-17T07:04:55+08:00"
completed_at: "2026-06-17T07:05:05+08:00"
source_root: "/home/hyde/workspace/local_backtest"
target_root: "/home/hyde/workspace/quant-lab"
content_read: false
checksum_or_sample_read: false
delete_executed: false
---

# CR080 Data/reports Restore Execution

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CR080 CP2/CP3/CP5 approved | PASS | `process/checkpoints/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md` / `process/checkpoints/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-REVIEW.md` / `process/checkpoints/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md` | 用户回复“同意”，批准进入 post-approval preflight 和受控 copy 条件。 |
| Preflight 通过 | PASS | `process/checks/CR080-DATA-REPORTS-RESTORE-PREFLIGHT-2026-06-17.md` | target `reports/` 和 `data/` 不存在，源目录存在，denylist 命中 0，可用空间充足。 |
| 执行命令族符合授权 | PASS | `rsync -a --stats` with denylist excludes | 未使用 `--delete`，未使用 checksum，未打印逐文件列表。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | 执行前目标冲突复检 | PASS | shell guard: target missing or empty | 执行前再次检查 target `reports/`、`data/` 不存在或为空。 |
| 2 | `reports/` copy | PASS | `rsync -a --stats` | Number of files=`898`；regular transferred=`772`；deleted=`0`；total transferred file size=`4,529,789,958 bytes`。 |
| 3 | `data/` copy | PASS | `rsync -a --stats` | Number of files=`18`；regular transferred=`8`；deleted=`0`；total transferred file size=`61,723,552 bytes`。 |
| 4 | denylist exclude 参数保留 | PASS | command family evidence | `.env*`、token、secret、password、private、credential、`.key`、`.pem`、`id_rsa*`、`id_ed25519*` 均作为 exclude。 |
| 5 | 禁止删除边界 | PASS | rsync stats | Number of deleted files=`0`；未使用 `--delete`。 |
| 6 | 禁止内容读取边界 | PASS | execution mode | 未运行 checksum、sample print、parquet/csv/json parsing、provider/lake/catalog、NAS compare 或 runtime。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| `reports/` 已复制到 target root | PASS | `/home/hyde/workspace/quant-lab/reports/` | 复制完成，后续由 CP7 做 metadata-only verification。 |
| `data/` 已复制到 target root | PASS | `/home/hyde/workspace/quant-lab/data/` | 复制完成，后续由 CP7 做 metadata-only verification。 |
| 未执行不授权事项 | PASS | 本文件 Checklist | 未执行内容读取、删除、NAS/provider/lake/catalog、remote Git、runtime、CR046 recovery 或旧根退役。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Preflight evidence | `process/checks/CR080-DATA-REPORTS-RESTORE-PREFLIGHT-2026-06-17.md` | PASS | Copy 前置检查。 |
| Execution evidence | `process/checks/CR080-DATA-REPORTS-RESTORE-EXECUTION-2026-06-17.md` | PASS | 本文件。 |
| Target `reports/` | `/home/hyde/workspace/quant-lab/reports/` | PASS | 本地 target root 目录。 |
| Target `data/` | `/home/hyde/workspace/quant-lab/data/` | PASS | 本地 target root 目录。 |

## 结论

- 结论：`PASS`
- 阻断项：无。
- 豁免项：无。
- 下一步：进入 CP7 metadata-only verification；验证只允许统计源/目标文件数、目录数、字节数、denylist count 和 `rsync -an --stats` dry-run，不允许内容读取或 checksum。
