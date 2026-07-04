# CR067 Clean Publication Execution Plan

## 1. 执行目标

CR067 目标是在不发布当前分支既有 tracked history 的前提下，建立一个可审计、可扫描、可回退的 clean GitHub publication baseline。

当前文件只是执行计划，不授权任何命令。`execute_allowed_now=false`；只有 CR067 CP2 / CP3 / CP5 均 approved 后，才允许进入 post-approval execution。

## 2. 推荐方案

推荐采用 `temporary clean snapshot + allowlist manifest + fail-closed scan + exact remote push`：

1. 从当前工作区读取已批准 allowlist 文件内容，构造临时 clean snapshot。
2. 不复制 `data/**`、`reports/**`、`runs/**`、`.env`、`.env.local`、`.envrc`、`.venv/**`、`__pycache__/**`、`*.pyc`、`process/STATE.md`、full process history、NAS / internal topology 文件。
3. 对 snapshot 执行 no-secret、data body、large-file、internal topology、path boundary 扫描。
4. 扫描任一 BLOCKING / HIGH finding 时 fail-closed，不执行 Git add / commit / remote add / push。
5. 扫描通过后，在临时 snapshot 内初始化独立 Git repo，提交 clean baseline，并只向 exact target `git@github.com:hyde-zhao/quant-lab.git` 推送。
6. 不 rewrite 当前仓库历史，不 force push，不删除 remote，不 tag，不恢复 CR046。

## 3. 方案对比

| 方案 | 优点 | 代价 / 风险 | 推荐结论 |
|---|---|---|---|
| A. current branch as-is push | 最快 | 会发布 tracked `reports/**` / `runs/**` / `.env.example` 与完整历史；已被 CR062 阻断 | 不采用 |
| B. temporary clean snapshot | 风险可控；不改当前分支；可扫描后发布 | 需要临时复制 allowlist 文件，GitHub 历史从 clean baseline 开始 | 推荐 |
| C. clean branch in current repo | 保留同仓操作 | 需要 branch creation，容易与当前工作树混淆 | 备选 |
| D. orphan branch | 同仓 clean history | branch / remote 默认分支治理复杂 | 备选 |
| E. history rewrite | 可清理历史 | 高风险，不可逆，需 force push | 不采用 |

## 4. Allowlist Policy

### 必选 include

| 路径 | 理由 |
|---|---|
| `README.md` | 项目入口。 |
| `docs/USER-MANUAL.md` | 用户说明。 |
| `pyproject.toml` | 项目依赖和 Python 版本声明。 |
| `uv.lock` | 锁定依赖。 |
| `quant_lab/**` | quant-lab compatibility namespace。 |
| `engine/**` | legacy engine root；`quant_lab.engine` 依赖它。 |
| `market_data/**` | legacy market_data root；CLI / readers / contracts 依赖它。 |
| `strategies/**` | legacy strategy root。 |
| `trading/**` | static-only source boundary；runtime 不执行。 |
| `tests/conftest.py` | pytest 基础配置。 |
| `tests/test_cr061_package_import_layout.py` | CR061 package/import smoke baseline。 |

### 条件 include

| 路径 | 条件 |
|---|---|
| `.env.example` | placeholder-only / no-secret scan PASS，且 CP5 DQ 明确接受。 |
| selected `docs/release/*CR061*` / `*CR066*` | public-safe、无内网拓扑、无真实 lake root、体积受控。 |
| selected `process/changes/CR-061/062/066/067*.md` | 仅保留必要审计摘要；不得纳入 full process history。 |
| selected checkpoint launch / final files | 仅 public-safe 且无 internal topology。 |

### 默认 exclude

| 路径 | 理由 |
|---|---|
| `.env`, `.env.local`, `.envrc` | 凭据 / runtime 配置边界。 |
| `data/**` | 数据体不发布。 |
| `reports/**` | 研究报告 / 产物体不发布。 |
| `runs/**` | 运行记录不发布。 |
| `.venv/**` | 本地环境不发布。 |
| `**/__pycache__/**`, `**/*.pyc` | 缓存产物不发布。 |
| `process/STATE.md` | 长期过程状态、体积大、含内部恢复信息。 |
| full `process/checks/**`, full `process/checkpoints/**`, full `process/context/**` | 过程历史不全量公开。 |
| `ops/systemd/**`, NAS mount docs, internal topology docs | 可能暴露内网路径 / 拓扑。 |
| `notebooks/**` | 未执行内容和元数据未做发布审计。 |

## 5. Post-Approval Execution Order

CR067 CP2 / CP3 / CP5 approved 后才允许执行以下顺序。任一 stop condition 命中即停止。

1. 重新读取 CR067 CP5 checkpoint，确认 `approved`。
2. 重新确认 `execute_allowed_now=false` 已被 CP5 approval 转为 post-approval scope，不代表无限授权。
3. 冻结 manifest 到 `docs/release/CLEAN-PUBLICATION-EXECUTION-MANIFEST-CR067.yaml`。
4. 创建临时 clean snapshot 目录，例如 `/tmp/quant-lab-publication-cr067-<timestamp>`。
5. 按 allowlist 复制文件到 snapshot，复制时跳过默认 exclude 和缓存产物。
6. 对 snapshot 执行 no-secret、data body、large-file、internal topology、path boundary 扫描。
7. 若 `.env.example` 存在，单独验证 placeholder-only；失败则从 snapshot 删除并记录。
8. 扫描 PASS 后，在 snapshot 内初始化 Git repo、创建 clean baseline commit、添加 exact remote、推送到 exact target。
9. push 成功或失败都写入执行证据；失败不 force push、不删除 remote、不 rewrite history。
10. 进入执行后验证 / release readiness，单独生成后续 CP6 / CP7 / CP8 或 blocked evidence。

## 6. Stop Conditions

| 条件 | 处理 |
|---|---|
| 任一 scan 出现 BLOCKING / HIGH | 停止，不执行 Git add / commit / remote add / push。 |
| `.env.example` 含疑似真实 token/password/cookie/session/private key | 从 snapshot 排除；必要时重开 DQ。 |
| allowlist 解析出不存在或超范围路径 | 停止，更新 manifest。 |
| snapshot 含 `data/**`、`reports/**`、`runs/**`、`.env*` 非例外文件 | 停止，清理 snapshot 并记录。 |
| remote 已存在且 target 不一致 | 停止，询问用户。 |
| remote push 被拒绝或目标仓已有不兼容历史 | 停止，forward-fix；不 force push。 |
| 需要 branch rename / default branch rename / tag / force push / remote deletion | 停止，新建 DQ 或 CR。 |

## 7. 回滚 / Forward-Fix

- 当前仓库不被 rewrite；回滚主要是删除或忽略临时 snapshot。
- 若未 push，删除临时 snapshot 即可回到 CR062 blocked baseline。
- 若 push 部分成功但后续验证失败，不 force push、不删除 remote；创建后续修复 CR 或在远端新增修复 commit。
- 若 remote conflict，保持 CR067 blocked 并记录 exact error，不自动 set-url 或 remote deletion。

## 8. 不授权项

本文件不授权任何实际命令。CP5 approval 前仍不授权 `git add`、`git commit`、`git remote add`、`git push`、`git tag`、branch creation / rename、history rewrite、force push、remote deletion、NAS/data lake/provider/catalog/runtime、凭据读取、physical move、bulk import rewrite 或 CR046 recovery。
