---
status: "ready"
change_id: "CR-071"
release_decision: "READY_WITH_RISK"
created_at: "2026-06-16T09:13:26+08:00"
---

# CR071 Deploy Checklist

## 部署类型

CR071 没有服务部署、安装包发布或运行时切换。本文件记录本机 shadow root 的交付就绪检查。

## 检查矩阵

| 检查项 | 结果 | 证据 |
|---|---|---|
| source root 存在 | PASS | `/home/hyde/workspace/local_backtest` |
| target root 存在 | PASS | `/home/hyde/workspace/quant-lab` |
| 旧目录保留 | PASS | 未删除、未重命名、未移动旧目录。 |
| target 排除项不存在 | PASS | target 根目录未发现 `reports/ data/ .venv/ node_modules/ .env`。 |
| target 排除项未被 Git 跟踪 | PASS | `git status --short -- reports data .venv node_modules .env` 与 `git ls-files ...` 均为空。 |
| branch 一致 | PASS | 两个根均为 `work/chapter3-factor-gap-remediation-20260608`。 |
| remote master 只读核对 | PASS | `3ade165e8b1edad031a911490cf6c1cee942616e refs/heads/master`。 |
| remote 写入 | PASS | 未执行。 |
| runtime / data lake 操作 | PASS | 未执行。 |

## 跳过项

| 项目 | 结论 | 原因 |
|---|---|---|
| `uv sync` / `.venv` 重建 | skipped | `.venv` 明确排除且 CR071 不授权运行环境重建。 |
| full pytest | skipped | 目标根不是完整 runtime/data mirror，且本轮验证目标是 copy boundary。 |
| NAS 迁移验证 | covered-by-CR072 | 归档已由 CR072 单独执行并校验。 |
| Git push 验证 | skipped | 远端 Git 写动作不授权。 |
| QMT / MiniQMT runtime 验证 | skipped | runtime 操作不授权。 |

## 放行结论

`READY_WITH_RISK`。可关闭 CR071 当前本机 copy-first 交付；后续 root cutover、环境重建、数据/报告恢复、远端同步或 runtime 操作必须单独授权。

