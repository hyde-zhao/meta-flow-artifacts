---
status: "pending-cp2-cp3-cp5-approval"
version: "1.0"
change_id: "CR-071"
title: "Copy-first Shadow Root Migration Plan"
created_by: "host-orchestrator"
created_at: "2026-06-16T07:16:44+08:00"
execute_allowed_now: false
---

# CR071 Copy-first Shadow Root Migration Plan

## 1. 目标

在本机创建 `/home/hyde/workspace/quant-lab` 作为 `/home/hyde/workspace/local_backtest` 的 shadow root。迁移策略是 copy-first：先复制到新目录，旧目录保持原样；验证通过前不切换日常工作根、不删除旧目录、不重命名旧目录。

## 2. 范围

| 项目 | 决策 |
|---|---|
| Source root | `/home/hyde/workspace/local_backtest` |
| Target root | `/home/hyde/workspace/quant-lab` |
| Root relationship | 同级目录；不是 repo 内子目录 |
| Old root | 保留，不删除、不重命名、不移动 |
| Copy mode | `rsync -a` copy-first |
| `.git` | 推荐复制，用于保留本地 Git 工作树和历史；不执行远端 Git 写动作 |
| `.env` | 默认不复制 |
| Excluded directories | `reports/`、`data/`、`.venv/`、`node_modules/` |
| Included tracked docs/code/process | 默认复制，除上述排除项外保持工作树形态 |

## 3. 执行前置条件

| 条件 | 状态 | 失败行为 |
|---|---|---|
| CP2 approved | pending | 不执行 |
| CP3 approved | pending | 不执行 |
| CP5 approved | pending | 不执行 |
| `/usr/bin/rsync` 可用 | observed | 若执行时不可用，阻断 |
| target root 不存在或为空 | observed-currently-absent | 若执行时非空，阻断并重提决策 |
| `.env` 不纳入复制 | planned | 若用户要求复制，必须重提 CP5 |
| data/reports 不纳入复制 | planned | 若用户要求复制，必须另起 data/artifact migration CR |

## 4. 执行命令

```bash
mkdir -p /home/hyde/workspace/quant-lab
rsync -a \
  --exclude '/reports/' \
  --exclude '/data/' \
  --exclude '/.venv/' \
  --exclude '/node_modules/' \
  --exclude '/.env' \
  /home/hyde/workspace/local_backtest/ \
  /home/hyde/workspace/quant-lab/
```

## 5. 验证策略

| 验证项 | 方法 | 预期 |
|---|---|---|
| Target root exists | `test -d /home/hyde/workspace/quant-lab` | PASS |
| Old root retained | `test -d /home/hyde/workspace/local_backtest` | PASS |
| Excluded directories absent | `test ! -e target/reports` 等 | PASS |
| `.env` absent | `test ! -e /home/hyde/workspace/quant-lab/.env` | PASS |
| Git metadata copied | `test -d /home/hyde/workspace/quant-lab/.git` | PASS unless CP5 修改 |
| Target Git status readable | `git -C /home/hyde/workspace/quant-lab status --short` | PASS；不执行 remote |
| Python metadata readable | `test -f pyproject.toml && test -f uv.lock` | PASS |
| CR071 manifest copied | `test -f docs/release/COPY-FIRST-SHADOW-ROOT-MANIFEST-CR071.yaml` | PASS |

## 6. 回退策略

copy-first 的回退默认是继续使用旧 root。若目标目录复制失败或验证失败，不自动删除 `/home/hyde/workspace/quant-lab`，先记录 partial-copy 状态并回到人工决策；删除目标目录需要用户独立授权。

## 7. 不授权项

- 不删除、不重命名、不移动 `/home/hyde/workspace/local_backtest`。
- 不复制 `reports/`、`data/`、`.venv/`、`node_modules/` 或 `.env`。
- 不读取 `.env` 或任何凭据正文。
- 不执行 NAS、远端 Git 写动作、QMT / MiniQMT runtime、provider fetch、lake write、catalog publish 或数据湖 root 切换。
- 不自动切换 shell、IDE、cron、systemd、外部脚本或环境变量到新 root。

## 8. 后续

CR071 执行完成后，后续是否把日常工作根切到 `/home/hyde/workspace/quant-lab`、是否处理 `reports/` / `data/`、是否调整 data lake root、是否做 NAS 或远端 Git 操作，均不从 CR071 自动继承授权。
