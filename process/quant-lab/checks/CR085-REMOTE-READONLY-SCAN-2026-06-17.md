---
check_id: "CR085-REMOTE-READONLY-SCAN-2026-06-17"
type: "remote_readonly_scan"
status: "PASS_WITH_RISK"
owner: "host-orchestrator"
created_at: "2026-06-17T13:04:40+08:00"
checked_at: "2026-06-17T13:04:40+08:00"
boundary_corrected_at: "2026-06-17T13:34:20+08:00"
source_cr: "CR-085"
parent_cr: "CR-084"
remote_url: "git@github.com:hyde-zhao/quant-lab.git"
remote_branch: "master"
remote_head: "3ade165e8b1edad031a911490cf6c1cee942616e"
local_candidate_commit: "d4d2881"
env_example_policy: "allowed-template-retain"
env_policy: "forbidden-upload-no-content-read"
---

# CR085 Remote Readonly Scan

## Entry Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 用户确认目标远端 | PASS | 结构化选择：quant-lab GitHub | 目标为 `git@github.com:hyde-zhao/quant-lab.git`。 |
| 用户选择扫描策略 | PASS | 结构化选择：先扫描清单 | 只读扫描，不执行远端写入或删除。 |
| CR084 已批准但范围不足 | PASS | `process/changes/CR-084-REMOTE-TARGET-CONFIGURATION-PUSH-EXECUTION-GATE-2026-06-17.md` | CR084 只覆盖远端目标配置和普通 push，不覆盖远端清理策略。 |

## Checklist

| # | 检查项 | 状态 | 证据 | 处理意见 |
|---|---|---|---|---|
| 1 | 远端 HEAD / 分支 | PASS | `git ls-remote --symref git@github.com:hyde-zhao/quant-lab.git HEAD refs/heads/*` | HEAD 指向 `refs/heads/master`；master 为 `3ade165e8b1edad031a911490cf6c1cee942616e`。 |
| 2 | 临时只读扫描 clone | PASS | `/tmp/cr085-remote-scan-quant-lab-master-20260617T1248` | 使用 `--filter=blob:none --no-checkout --single-branch --branch master`；只读远端，不改当前仓库 remote。 |
| 3 | 远端文件数量 | PASS | `git -C /tmp/cr085-remote-scan-quant-lab-master-20260617T1248 ls-tree -r --name-only HEAD` | remote master 当前 136 个 tracked 文件。 |
| 4 | 本地候选 commit 文件数量 | PASS | `git ls-tree -r --name-only d4d2881` | 本地候选 `d4d2881` 当前 570 个 tracked 文件。 |
| 5 | 远端禁止路径命中 | PASS | `git -C /tmp/... ls-tree -r --name-only HEAD reports runs data .env .env.example process` | 用户已确认 `.env.example` 可以保留；按修正边界，远端未见 `.env`、`reports/`、`runs/`、`data/`、`process/`。 |
| 6 | 本地候选禁止路径命中 | FAIL | `git ls-tree -r --name-only d4d2881 reports runs data .env .env.example process` | `.env.example` 可以保留；仍命中 `runs/RUN-EXEC-20260613-001.md`，且 `d4d2881` 会把远端 tracked 文件面从 136 扩到 570，因此不能直接 push。 |
| 7 | 远端写动作 | PASS | 本轮命令记录 | 未执行 `git remote add`、set-upstream、push、force、tag、history rewrite 或远端删除。 |

## Exit Criteria

| 条目 | 状态 | 证据 | 说明 |
|---|---|---|---|
| 远端现状可判定 | PASS | 本扫描 + 用户修正 | `.env.example` 是允许保留模板；远端当前未见 `.env`、`reports/`、`runs/`、`data/`、`process/`，当前没有因 `.env.example` 产生的删除项。 |
| 直接推送 `d4d2881` 是否安全 | FAIL | 本扫描 | `d4d2881` 仍包含 `runs/RUN-EXEC-20260613-001.md`，且会扩大远端文件面。 |
| 下一步是否需要新门禁 | PASS | CR085 | 需要 CP2/CP3/CP5 决策：`.env` 禁止上传、clean publication commit / branch、是否需要任何远端删除或历史清理。 |

## Deliverables

| 交付物 | 路径 | 状态 | 说明 |
|---|---|---|---|
| 只读扫描证据 | `process/checks/CR085-REMOTE-READONLY-SCAN-2026-06-17.md` | PASS_WITH_RISK | 本文件。 |
| CR085 formal CR | `process/changes/CR-085-REMOTE-SYNCHRONIZATION-CLEANUP-GATE-2026-06-17.md` | PENDING | 用于承接远端同步与删除策略。 |

## 结论

- 结论：`PASS_WITH_RISK`
- 阻断项：不得直接 push `d4d2881` 到远端 master；必须先定义 clean publication / cleanup 策略。
- 用户边界修正：`.env.example` 可以保留；`.env` 禁止上传。本轮未读取 `.env` 内容。
- 远端当前待删除候选：无。当前扫描未发现 `.env`、`reports/`、`runs/`、`data/`、`process/`；`.env.example` 不再列为删除对象。
- 本地候选提交中仍需排除：`runs/RUN-EXEC-20260613-001.md`；后续 clean publication 还必须继续排除 `.env`、`reports/**`、`data/**`、`process/**` 等未授权边界。
- 推荐下一步：启动 CR085 CP2/CP3/CP5，确认 `.env` 禁止上传和 clean publication 策略；远端删除 / 历史清理 / force push 只能作为独立高风险选项，不默认执行。
