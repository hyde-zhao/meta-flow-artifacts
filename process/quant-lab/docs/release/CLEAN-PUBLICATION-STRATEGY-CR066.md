# CR066 Clean Publication Strategy

## 推荐方案

推荐采用 `clean snapshot / allowlist manifest` 作为下一步策略：

1. 不发布当前分支既有 history。
2. 不在 CR066 中创建分支、重写历史或 push。
3. 先冻结 clean source baseline allowlist。
4. 后续另起 CR067 Clean Publication Execution Gate，再决定是否创建 clean branch / temporary export / orphan snapshot / new repository import。

## 方案对比

| 方案 | 优点 | 代价 / 风险 | CR066 推荐 |
|---|---|---|---|
| A. current branch as-is publish | 最快 | 会发布 tracked `reports/**` / `runs/**` / `.env.example` 以及完整历史；违反 CR062 exclude policy | 不推荐 |
| B. clean snapshot / allowlist manifest | 风险最可控；避免发布历史污染；可审计 | 需要后续执行 CR；需要精确 allowlist 和扫描 | 推荐 |
| C. orphan branch / clean branch publish | 可在同仓库发布干净基线 | branch creation / branch push 需单独授权；默认分支治理需要决策 | 候选，需要 CR067 |
| D. history rewrite | 可从历史中移除污染 | 高风险、不可逆、需 force push 或重写本地历史；当前明确不授权 | 不推荐，除非未来独立 CR |
| E. private repository first | 外部暴露风险低 | 不满足 public baseline 目标；仍需 remote 权限策略 | 备选 |

## Clean Manifest Policy

### 默认 include 候选

- `README.md`
- `docs/USER-MANUAL.md`
- `pyproject.toml`
- `uv.lock`
- `quant_lab/**`
- CR061 package/import/layout tests
- 必要的公开安全边界文档

### 默认 exclude

- `.env`
- `.env.local`
- `.envrc`
- `data/**`
- `reports/**`
- `runs/**`
- `.venv/**`
- `**/__pycache__/**`
- `**/*.pyc`
- `process/STATE.md`
- 历史 process/checks/checkpoints 全量
- NAS mount / systemd / internal topology 文件

### 条件 include

| 路径 | 条件 |
|---|---|
| `.env.example` | 只含占位符；无真实 token/password/cookie/session/private key；人工批准例外。 |
| CR058-CR066 scoped process evidence | 仅公开所需、体积受控、无内部拓扑或已 redacted。 |
| release docs | 不含内网 IP、NAS export、真实 lake root、凭据、账户或交易事实。 |

## 后续执行 CR 边界

CR066 不执行实际 publish。若 CP5 / CP8 批准，应启动 CR067 或等价执行 CR，且至少包含：

- clean manifest freeze
- no-secret scan
- data/report/run path scan
- large file scan
- internal topology scan
- Git operation plan
- remote conflict policy
- rollback / forward-fix policy
- 用户人工 runtime_authorization

执行 CR 未批准前仍不得 `git add`、`git commit`、`git remote add`、`git push`、branch creation / rename、history rewrite 或 force push。
