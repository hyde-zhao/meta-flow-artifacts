# CR072 Verification Report

## 结论

`PASS_WITH_RISK`。

CR072 已按用户“按照你的建议处理”的授权完成本地资产治理：

- `reports/` 已归档到 NAS，源 / 目标 SHA256 一致。
- `data/` 已归档到 NAS，源 / 目标 SHA256 一致。
- `docs/`、`process/`、`ops/` 过程证据已归档到 NAS `evidence-tree/`，源 / 目标 SHA256 一致。
- `.venv/`、`node_modules/`、`.pytest_cache/` 和 `__pycache__/` 已作为可重建缓存清理。
- 本地 `reports/` 与 `data/` 均保留。
- 未执行远端 Git 写入、data lake 切换、QMT / MiniQMT runtime、凭据读取或 CR046 recovery。

## 验证矩阵

| 检查项 | 结果 | 证据 |
|---|---|---|
| reports NAS 归档 | PASS | `/mnt/quant-lab-primary/research/reports/cr072-local-backtest-reports-20260616T081950/reports/`，772 files |
| reports SHA256 | PASS | `reports-source-sha256.txt` 与 `reports-dest-sha256.txt` 一致 |
| data NAS 归档 | PASS | `/mnt/quant-lab-primary/data-lake/snapshots/cr072-local-backtest-data-20260616T081950/data/`，8 files |
| data SHA256 | PASS | `data-source-sha256.txt` 与 `data-dest-sha256.txt` 一致 |
| evidence NAS 归档 | PASS | `/mnt/quant-lab-primary/archive/migration-evidence/cr072-local-assets-evidence-20260616T081950/evidence-tree/`，2,091 files |
| evidence SHA256 | PASS | `evidence-source-sha256.txt` 与 `evidence-dest-sha256.txt` 一致 |
| 本地 reports 保留 | PASS | `reports/` 仍有 772 files，约 4.3G |
| 本地 data 保留 | PASS | `data/` 仍有 8 files，约 59M |
| 可重建缓存清理 | PASS | `.venv/`、`node_modules/`、`.pytest_cache/` 不存在；`__pycache__` count = 0 |
| 远端 Git 边界 | PASS | 远端 `master` 仍为 `3ade165e8b1edad031a911490cf6c1cee942616e` |

## 风险和说明

- `reports/` 首次 `rsync -a` 在 NFS 上因 group 元数据保留失败返回 code 23；内容已传输完整，随后用不保留 owner/group/perms 的参数重跑返回 code 0，并通过 SHA256 校验。
- NAS evidence root 中保留一个 `evidence-flat-initial-copy/`，这是初次复制时的平铺副本；正式审计入口是 `evidence-tree/`，最终同步后包含 2,091 个文件。
- 本轮清理了 `.venv/`，后续运行 Python 命令前需要用 `uv sync` 或 `uv run` 重建环境。

## 不授权项复核

- 未删除本地 `reports/`。
- 未删除本地 `data/`。
- 未 push、tag、改 branch、改 default branch、force push 或 history rewrite。
- 未切换 data lake root。
- 未连接 QMT / MiniQMT runtime。
- 未读取凭据。
- 未恢复 CR046。
