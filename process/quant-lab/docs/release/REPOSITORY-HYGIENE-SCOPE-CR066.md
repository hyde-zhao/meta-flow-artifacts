# CR066 Repository Hygiene Scope

## 目标

CR066 只定义 `quant-lab` clean publication strategy，用于处理 CR062 post-approval preflight 发现的 tracked forbidden paths。

本 CR 不执行 Git 写动作，不创建或切换分支，不做历史改写，不添加 remote，不 push。

## In Scope

| 类别 | 处理 |
|---|---|
| tracked `reports/**` | 评估是否允许出现在 public baseline、是否应从未来 clean manifest 排除，或是否需要独立风险接受。 |
| tracked `runs/**` | 评估是否属于运行态证据，默认从 public baseline 排除。 |
| `.env.example` | 评估是否作为占位示例文件例外允许；必须通过 placeholder / no-secret scan。 |
| clean publication route | 比较 current-branch publish、clean snapshot、orphan branch、history rewrite、private repo 等方案。 |
| follow-up execution boundary | 定义 CR067 或等价后续执行门禁，确保执行前另行批准。 |

## Out of Scope

| 项目 | 说明 |
|---|---|
| `git add` / `git commit` / `git remote add` / `git push` | CR066 不授权。 |
| `git tag` | 不授权。 |
| branch creation / rename / default branch rename | 不授权。 |
| history rewrite / force push / remote deletion | 不授权。 |
| NAS/data lake/provider/catalog/runtime/凭据 | 不授权读取、写入或运行。 |
| physical move / rename / delete | 不授权。 |
| bulk import rewrite | 不授权。 |
| CR046 recovery | 不恢复。 |

## Recommended Boundary

CR066 推荐只批准策略：

1. 暂停 CR062 current-branch publication。
2. 选择 clean publication route，优先 `clean snapshot / allowlist manifest`，但执行另起 CR。
3. `.env.example` 可作为示例文件候选，但必须通过 no-secret placeholder scan。
4. tracked `reports/**` / `runs/**` 默认不进入 public source baseline，若用户接受风险必须单独 DQ。
