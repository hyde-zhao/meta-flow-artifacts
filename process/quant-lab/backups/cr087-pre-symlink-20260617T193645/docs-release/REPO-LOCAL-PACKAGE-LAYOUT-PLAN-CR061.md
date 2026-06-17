---
status: "draft-for-cp5-review"
version: "1.0"
change_id: "CR-061"
title: "CR061 Repo-local Package Layout Plan"
created_by: "host-orchestrator"
created_at: "2026-06-15T07:29:19+08:00"
execute_allowed: false
---

# CR061 Repo-local Package Layout Plan

## 1. 推荐架构

推荐采用 staged compatibility-first，而不是一次性把全部源码移动到 `src/quant_lab/**`。

| 层级 | 推荐方案 | 说明 |
|---|---|---|
| Distribution metadata | 后续 CP5 批准后将 `[project].name` 收敛为 `quant-lab` | 必须用 `uv lock` 生成 lock diff，不手工改 lock。 |
| Legacy import roots | 保留 `engine` / `market_data` / `strategies` / `trading` | 现有测试、脚本、用户命令依赖这些入口。 |
| New namespace | 后续实现可新增 `quant_lab` compatibility namespace | 只做 re-export / alias bridge，不改变旧入口行为。 |
| CLI | 保留 `python -m market_data.cli` 等现有入口 | 新 CLI 或 console scripts 需要独立设计，不在当前门禁直接执行。 |
| Bulk move | 延后为二次授权候选 | 需要 clean/explicit dirty worktree manifest、fresh rollback_ref、full offline regression。 |

## 2. 候选方案对比

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. Staged compatibility-first | 风险最低；保留现有测试和用户命令；可逐步引入 `quant_lab` | 旧 root 会在兼容期继续存在 | `pyproject.toml`、`uv.lock`、新增 namespace、smoke tests、docs | 推荐 | 当前已有大量顶层 imports，且 CR046 暂停不应被 package move 触发。 |
| B. Bulk move to `src/quant_lab/**` | 目录最整洁；长期 package 形态明确 | 需要大规模 import rewrite；回滚复杂；容易破坏脚本和测试 | 295 个 Python 文件、测试、脚本、CLI、文档命令 | 不推荐作为第一步 | 仅在 full dry-run、clean worktree、fresh bundle 和用户二次授权后切换。 |
| C. Keep legacy roots forever | 零迁移风险；测试稳定 | 无法完成 quant-lab package identity convergence | 文档和 Git remote 可先完成，但 package 层仍旧名 | 备选保守 | 若短期优先远端发布、放弃 package rename，可选择。 |

## 3. 执行前置条件

| gate_id | 前置条件 | 当前结果 | 说明 |
|---|---|---|---|
| PLG-CR061-001 | CR061 CP2 approved | PENDING | 需求 / 范围未人工确认。 |
| PLG-CR061-002 | CR061 CP3 approved | PENDING | staged architecture 未人工确认。 |
| PLG-CR061-003 | CR061 CP5 approved | PENDING | candidate list / rollback / no-runtime 未人工确认。 |
| PLG-CR061-004 | rollback_ref refreshed or explicitly waived | BLOCKED | 当前复用 CR059 bundle 仅适合 planning；真实实现前需刷新或风险接受。 |
| PLG-CR061-005 | dirty worktree include/exclude manifest | BLOCKED | 当前工作树含 CR058/059/060/061 文档和 README/USER-MANUAL 改动。 |
| PLG-CR061-006 | full offline regression plan | PENDING | 后续 CP6/CP7 需要明确 pytest subset/full run。 |
| PLG-CR061-007 | runtime / provider / lake disabled | PASS | 当前设计显式禁止。 |

## 4. 实现批次草案

| Batch | 目标 | 可执行条件 | 禁止项 |
|---|---|---|---|
| Batch A | CR061 门禁与设计冻结 | 当前用户启动 CR061 | 不改代码。 |
| Batch B | package metadata + compatibility namespace | CP5 approved + rollback_ref + dirty manifest | 不 bulk move、不删除 legacy roots。 |
| Batch C | docs/test smoke refresh | Batch B 完成 + CP6 evidence | 不读 `.env`、不运行 provider / lake / runtime。 |
| Batch D | bulk physical relayout | 二次执行授权 + full dry-run + fresh bundle | 未授权前禁止。 |

## 5. 回归策略

| 层级 | 验证命令 / 方法 | 当前状态 |
|---|---|---|
| TOML / lock | `uv lock` 后 diff review | 未执行。 |
| Syntax | `uv run --python 3.11 python -m py_compile ...` | 后续选择 subset。 |
| Import smoke | `python -c "import engine, market_data, strategies, trading"` | 后续执行。 |
| New namespace smoke | `python -c "import quant_lab"` | 仅实现后执行。 |
| Offline pytest | `uv run --python 3.11 pytest -q` 或风险化 subset | 后续 CP7。 |
| Forbidden runtime | static grep + tests with env blank | 后续 CP7；不得连接 provider / QMT。 |

## 6. 不授权项

- 不执行 `mv` / `rename` / `delete`。
- 不批量 rewrite imports。
- 不修改 `pyproject.toml` 或 `uv.lock`。
- 不创建 Git remote、不 push、不 tag、不 rename branch、不 rewrite history。
- 不复制 / 移动 / 删除 NAS 数据。
- 不替换 `MARKET_DATA_LAKE_ROOT`、不 provider fetch、不 lake write、不 catalog publish。
- 不读取 `.env`、token、password、cookie、session、private key、账户或交易事实。
- 不启动 QMT / MiniQMT runtime，不查询账户，不提交/撤销订单，不 simulation/live。
- 不恢复 CR046。
