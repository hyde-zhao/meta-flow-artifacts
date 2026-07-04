---
status: "PASS_WITH_RISK"
version: "1.0"
change_id: "CR-071"
title: "CR071 Copy-first Shadow Root Test Report"
created_by: "host-orchestrator"
created_at: "2026-06-16T07:33:48+08:00"
---

# CR071 Copy-first Shadow Root Test Report

## 1. 命令结果

| 命令 / 检查 | 结果 | 说明 |
|---|---|---|
| `test -d /home/hyde/workspace/local_backtest` | PASS | source root 存在。 |
| `test ! -e /home/hyde/workspace/quant-lab` | PASS before execution | 执行前 target 不存在。 |
| `command -v rsync` | PASS | `/usr/bin/rsync`。 |
| `mkdir -p /home/hyde/workspace/quant-lab` | PASS | 创建 target root。 |
| `rsync -a --exclude ...` | PASS | 实际 copy 完成。 |
| `test -d /home/hyde/workspace/quant-lab` | PASS | target exists。 |
| `test -d /home/hyde/workspace/local_backtest` | PASS | old root retained。 |
| excluded path checks | PASS | `reports/ data/ .venv/ node_modules/ .env` 均 absent。 |
| `test -d /home/hyde/workspace/quant-lab/.git` | PASS | `.git` copied。 |
| `git -C /home/hyde/workspace/quant-lab rev-parse HEAD` | PASS | `e7a3a0d7e66b3972b95c047f017790d9853625d0`。 |
| `git -C /home/hyde/workspace/quant-lab branch --show-current` | PASS | `work/chapter3-factor-gap-remediation-20260608`。 |
| `git -C /home/hyde/workspace/quant-lab status --short reports` | PASS_WITH_RISK at CP7 / mitigated before CP8 | CP7 时 tracked `reports/**` 显示 deleted，因为 reports excluded；CP8 前已通过停止 Git 跟踪 reports 缓解。 |

## 2. 未运行项

| 项目 | 原因 |
|---|---|
| Full pytest | CR071 是本机复制迁移，验证重点是 filesystem/result boundary；未授权依赖同步或运行环境重建。 |
| NAS validation | 本轮不授权 NAS。 |
| Remote Git validation | 本轮不授权远端 Git 写动作，也不需要远端检查。 |
| Runtime / data lake validation | 本轮不授权 QMT runtime、provider fetch、lake write 或 root switch。 |

## 3. 结论

`PASS_WITH_RISK`。复制行为成功，排除项正确；CP7 时 target 中 tracked `reports/**` 的 Git deleted 状态已在 CP8 前缓解。最终剩余风险是 target 不含运行/数据/报告环境，且 root cutover / old root retirement / runtime / data lake / remote Git 均未授权。

## 4. CP8 后续处置

| 项目 | 结果 | 说明 |
|---|---|---|
| reports Git tracking risk | mitigated-before-cp8 | source `45a426d`、target `2aaf21b` 已停止 Git 跟踪 reports，reports 本地文件保留。 |
| target excluded path status | PASS | `reports data .venv node_modules .env` 在 target 的 `git status`、`git ls-files` 和根目录查找均为空。 |
| CR072 archive | PASS | reports/data/process evidence 已归档到 NAS 并校验，本地 reports/data 保留。 |
