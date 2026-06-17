# CR062 Git Publication Scope

## 目标

CR062 只定义 `quant-lab` 当前 repo-local 成果发布到 GitHub 的准入范围。CP2 / CP3 / CP5 批准前，本文件不授权任何 Git 写动作。

推荐远端候选：`git@github.com:hyde-zhao/quant-lab.git`

## Include Policy

| 类别 | 推荐处理 | 说明 |
|---|---|---|
| 项目代码与 metadata | post-approval manifest 冻结后可纳入 | 包括 CR061 已验证的 `pyproject.toml`、`uv.lock`、`quant_lab/**`、相关测试。 |
| README / docs | 可纳入 | 用于 GitHub baseline 可读性。 |
| CR058-CR062 过程证据 | 可选择性纳入 | 只纳入迁移审计必需证据，不批量发布所有历史过程文件。 |
| release gate 文档 | 可纳入 | CR062 scope、manifest、scan plan、rollback 和 CP gate。 |

## Exclude Policy

| 路径 / 对象 | 处理 | 原因 |
|---|---|---|
| `.env` / `.env*` | 禁止读取、禁止提交 | 凭据边界。 |
| `data/**` | 禁止提交 | 数据湖 / 市场数据不属于 Git publication。 |
| `reports/**` | 默认禁止提交 | 可能包含运行输出、真实数据衍生物或大文件。 |
| `runs/**` | 默认禁止提交 | 运行产物不属于源码发布。 |
| `.venv/**` | 禁止提交 | 依赖环境不是交付物。 |
| `**/__pycache__/**` / `**/*.pyc` | 禁止提交 | 缓存产物不是交付物。 |
| `/mnt/**` / NAS mount | 禁止访问与提交 | NAS/data migration 不属于 CR062。 |
| trading runtime configs / account material | 禁止读取、禁止提交 | CR046 仍暂停，runtime 不授权。 |

## Gate Decision

CR062 CP5 approve 可授权后续按 manifest 执行受控 Git publication，但不授权 tag、branch rename、history rewrite、force push、remote deletion、NAS/data lake/provider/catalog/runtime/凭据、physical move 或 bulk import rewrite。
