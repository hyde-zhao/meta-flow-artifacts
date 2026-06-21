---
status: "current-with-cr108-addendum"
version: "1.1"
change_id: "CR-061"
feature_id: "cr061-package-import-layout"
created_at: "2026-06-15T08:41:42+08:00"
created_by: "host-orchestrator"
implementation_allowed: false
---

# CR061 Package / Import / Layout Feature Design

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-06-15 | host-orchestrator | 新增 Batch B 实现授权前设计，限定 metadata + compatibility namespace，不授权真实执行。 |
| 1.1 | 2026-06-21 | host-orchestrator | 按 CR-108 增补 current-truth addendum：CR061 后续实现已将 distribution metadata 和 `quant_lab` 兼容入口收敛到 `quant-lab`；原 Batch B 设计中的 `local-backtest` 回滚项保留为历史设计语境，不作为当前推荐状态。 |

## 0. CR108 Current-Truth Addendum

CR061 Batch B 的后续实现证据显示：`pyproject.toml` / `uv.lock` 已收敛为 `quant-lab`，并已新增 `quant_lab` 兼容命名空间。当前 package identity 真相源以 `docs/design/PACKAGE-IDENTITY.yaml`、`pyproject.toml` 和 `docs/features/cr061-package-import-layout/IMPLEMENTATION.md` 为准。

本文件的 1.0 设计内容保留为 CR061 Batch B 授权前设计历史；其中“恢复 `[project].name = "local-backtest"`”仅表示当时设计的回滚路径，不表示当前推荐项目身份。当前 CR108 不授权批量移动 legacy roots、不授权 bulk import rewrite、不授权 runtime、provider、lake、NAS、QMT 或凭据读取。

## 1. 问题定义

CR060 已完成 docs-only identity convergence，项目对外规范名收敛为 `quant-lab`，但 repo 内 Python distribution metadata 仍是 `local-backtest`，源码入口仍是 `engine` / `market_data` / `strategies` / `trading` 等顶层 legacy roots。直接移动目录或批量改 import 会触及测试、脚本、实验、CLI、交易边界和历史审计文件，风险面过大。

CR061 Batch B 的目标是给第一段可实现切片建立明确设计：只允许在后续人工门禁批准后做 package metadata 收敛、`uv.lock` 受控再生成、可选 `quant_lab` 兼容命名空间和离线 import smoke test。它不是 bulk relayout，也不是真实 Git/NAS/data lake/runtime 迁移。

## 2. 范围

| 类别 | 纳入 Batch B 候选 | 不纳入 |
|---|---|---|
| Project metadata | `pyproject.toml` `[project].name` 改为 `quant-lab` | 版本号、依赖策略、发布配置重构 |
| Lockfile | 通过 `uv lock` 再生成 `uv.lock` | 手工编辑 lock、换包管理器 |
| Package namespace | 新增最小 `quant_lab` 兼容入口 | 移动 legacy roots 到 `src/quant_lab/**` |
| Imports | 新 namespace smoke；legacy imports 保持可用 | bulk import rewrite、删除 legacy imports |
| Tests | 新增离线 import smoke test | provider、lake、NAS、QMT runtime 测试 |
| Docs | 仅在命令可见面变化时做最小更新 | 历史 process 证据批量改写 |

## 3. 推荐设计

推荐采用 compatibility-first：

1. `pyproject.toml` 的 distribution name 变更为 `quant-lab`，使项目 metadata 与 canonical project name 对齐。
2. `uv.lock` 只由 `uv lock` 或等价 `uv` 命令生成，不手工编辑。
3. 保留现有 legacy root：`engine`、`market_data`、`strategies`、`trading`。
4. 新增 `quant_lab` 作为兼容命名空间。实现可以从最小 `quant_lab/__init__.py` 开始，若后续源码检查确认低风险，再添加只做 module alias 的 `quant_lab.engine`、`quant_lab.market_data`、`quant_lab.strategies`、`quant_lab.trading`。
5. 通过离线 smoke test 同时证明 legacy imports 未破坏、新 namespace 可导入。

## 4. 备选方案

| Option | Pros | Cons | Impact Surface | Recommendation | Assumptions / When to switch |
|---|---|---|---|---|---|
| A. metadata + compatibility namespace | 小步收敛；可回滚；不破坏 legacy CLI；不触发 CR046 | 目录仍不是最终 `src/quant_lab` 形态 | `pyproject.toml`、`uv.lock`、新增 `quant_lab/**`、smoke test | 推荐 | 当前 dirty worktree 较大，且 package root 影响面需要先降风险。 |
| B. 只改 metadata | 最小改动；回滚容易 | 无法提供 `quant_lab` import 入口 | `pyproject.toml`、`uv.lock` | 备选保守 | 若源码 alias 设计存在循环 import 风险，先只执行 metadata。 |
| C. 直接 bulk relayout | 最终目录更整洁 | 需要大规模移动和 import rewrite；回滚复杂；触及 runtime 边界 | 源码、测试、脚本、实验、文档、历史证据 | 不推荐 | 仅在后续独立 CR 通过 clean worktree、fresh bundle、full dry-run 后切换。 |

## 5. 集成契约

| 对象 | 调用方向 | 输入契约 | 输出契约 | 降级策略 |
|---|---|---|---|---|
| `pyproject.toml` | uv / tooling 读取 | `[project].name = "quant-lab"` | distribution metadata 使用 canonical name | 若 lock diff 异常，回退 metadata 改动。 |
| `uv.lock` | uv / pytest 环境读取 | 由 `uv lock` 生成 | lock metadata 与 pyproject 对齐 | 若 lock 再生成引入非预期依赖 diff，停止并回退。 |
| `quant_lab` | 用户 / 测试 import | Python import only；不得触发 provider/lake/runtime | 可导入命名空间或 alias module | 若 alias 有副作用，只保留最小 namespace。 |
| legacy roots | 现有代码继续 import | `engine` / `market_data` / `strategies` / `trading` 保持原路径 | 现有导入不破坏 | 任何 legacy smoke 失败即回退 Batch B。 |
| smoke test | CP7 离线验证 | 不读 `.env`，不连接外部服务 | import 成功 / 失败证据 | 外部依赖导致失败时缩小为纯 import subset 并记录风险。 |

## 6. 前置校验与失败路径

| gate_id | 前置条件 | 失败行为 |
|---|---|---|
| FD-CR061B-001 | CP5 Batch B 人工门禁 approved | 未批准则不得改代码。 |
| FD-CR061B-002 | dirty worktree include/exclude 已冻结 | 未冻结则不得开始实现。 |
| FD-CR061B-003 | rollback_ref 刷新或风险接受 | 未满足则 BLOCKED，不执行。 |
| FD-CR061B-004 | 实现只命中 include paths | 命中 exclude path 则停止并返工门禁。 |
| FD-CR061B-005 | 验证命令不读取凭据、不连外部服务 | 命令需要凭据/runtime 时停止，另起授权。 |

## 7. 回滚策略

Batch B 回滚只允许回滚本批次 diff：

| 变更 | 回滚方式 |
|---|---|
| `pyproject.toml` metadata | 恢复 `[project].name = "local-backtest"`。 |
| `uv.lock` metadata | 用 pre-Batch-B diff 恢复 lock。 |
| `quant_lab/**` | 删除本批次新增兼容命名空间文件。 |
| smoke test | 删除或 revert 本批次新增测试文件。 |

不得通过历史改写、远端 force push、删除数据、改写 NAS 或改写历史 `process/**` 证据来回滚。

## 8. Gotchas

| gotcha | 规避方式 |
|---|---|
| `pyproject.toml` 名称改变不等于 import package 名称改变。 | 单独验证 distribution metadata 和 Python import namespace。 |
| `market_data.cli` 可能包含 provider / lake 写入路径。 | Batch B smoke 只做 import / `--help` 候选，不执行 normalize / publish。 |
| `trading/**` 与 CR046 暂停边界重叠。 | Batch B 只允许 static import；不恢复 CR046，不查询账户。 |
| `uv.lock` diff 可能包含环境解析变化。 | 只接受与 project name 相关的可解释 diff，否则停止。 |
| 历史审计文件保留旧名是正确行为。 | 不把 `process/**` 的 `local_backtest` 当成必须替换的缺陷。 |
