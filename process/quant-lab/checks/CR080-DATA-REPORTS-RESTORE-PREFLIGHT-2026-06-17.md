---
check_id: "CR080-DATA-REPORTS-RESTORE-PREFLIGHT"
cr_id: "CR-080"
status: "PASS"
owner: "host-orchestrator"
created_at: "2026-06-17T07:03:43+08:00"
checked_at: "2026-06-17T07:03:43+08:00"
source_root: "/home/hyde/workspace/local_backtest"
target_root: "/home/hyde/workspace/quant-lab"
content_read: false
checksum_or_sample_read: false
delete_authorized: false
copy_authorized_after_pass: true
---

# CR080 Data/reports Restore Preflight

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| CR080 CP2/CP3/CP5 已人工批准 | PASS | `process/checkpoints/CP2-CR080-DATA-REPORTS-RESTORE-BASELINE.md` / `process/checkpoints/CP3-CR080-DATA-REPORTS-RESTORE-DESIGN-REVIEW.md` / `process/checkpoints/CP5-CR080-DATA-REPORTS-RESTORE-READINESS.md` | 用户于 `2026-06-17T06:57:22+08:00` 回复“同意”。 |
| 仍遵守不授权边界 | PASS | `process/STATE.md.root_authority` / `process/changes/CR-080-DATA-REPORTS-RESTORE-COPY-GATE-2026-06-17.md` | 不读取内容、不解析样本、不删除、不访问 NAS/provider/lake/catalog/remote/runtime。 |
| 源目录存在 | PASS | metadata-only shell preflight | legacy retained assets 中 `reports/` 和 `data/` 均存在且不是 symlink。 |
| 目标目录不存在或为空 | PASS | metadata-only shell preflight | target root 下 `reports/`、`data/` 均不存在；不存在视为可创建。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | CR 台账一致性 | PASS | `uv run --python 3.11 python scripts/check_cr_tracking_consistency.py --project-root .` -> `CR tracking consistency: PASS` | 可继续。 |
| 2 | CP2 human gate 合规 | PASS | `scripts/check_human_gate_decision_brief.py` -> `decision_count=5` | 可继续。 |
| 3 | CP3 human gate 合规 | PASS | `scripts/check_human_gate_decision_brief.py` -> `decision_count=5` | 可继续。 |
| 4 | CP5 human gate 合规 | PASS | `scripts/check_human_gate_decision_brief.py` -> `decision_count=5` | 可继续。 |
| 5 | `rsync` 可用 | PASS | `command -v rsync` | 使用 `rsync -a`；不使用 `--delete`。 |
| 6 | 目标 root 可用 | PASS | `/home/hyde/workspace/quant-lab` exists and is not symlink | 可作为 authoritative target root。 |
| 7 | 源 `reports/` metadata | PASS | files=`772`, dirs=`126`, symlinks=`0`, bytes=`4529789958` | 只统计元数据，不读内容。 |
| 8 | 源 `data/` metadata | PASS | files=`8`, dirs=`10`, symlinks=`0`, bytes=`61723552` | 只统计元数据，不读内容。 |
| 9 | denylist 命中 | PASS | reports denylist paths=`0`; data denylist paths=`0` | 本次按 denylist 排除策略执行；命中为 0。 |
| 10 | 目标 `reports/` 状态 | PASS | target state=`missing` | 可创建；若执行中发现冲突立即停止记录。 |
| 11 | 目标 `data/` 状态 | PASS | target state=`missing` | 可创建；若执行中发现冲突立即停止记录。 |
| 12 | 可用空间 | PASS | source total bytes=`4591513510`; target available bytes=`969690243072`; fs=`ext4` | 可用空间大于源目录元数据总量。 |
| 13 | Whitespace 检查 | PASS | `git diff --check -- process/STATE.md process/changes/CR-INDEX.yaml ...` | 无输出，视为通过。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 无阻断项 | PASS | 本文件 Checklist | preflight 可放行受控本地 copy。 |
| 目标未发现非空冲突 | PASS | target reports/data missing | 符合 fail-closed 条件。 |
| 敏感路径排除策略可执行 | PASS | denylist paths = 0 | 即便命中为 0，copy 仍保留 denylist exclude 参数。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| Preflight evidence | `process/checks/CR080-DATA-REPORTS-RESTORE-PREFLIGHT-2026-06-17.md` | PASS | 本文件。 |

## 结论

- 结论：`PASS`
- 阻断项：无。
- 豁免项：无。
- 下一步：执行受控本地 copy：legacy retained assets `/home/hyde/workspace/local_backtest/reports/` 与 `/home/hyde/workspace/local_backtest/data/` 到 authoritative target root `/home/hyde/workspace/quant-lab/`。执行时不得使用 `--delete`，不得读取内容、不得打印文件清单、不得复制 credential-like denylist 路径。
