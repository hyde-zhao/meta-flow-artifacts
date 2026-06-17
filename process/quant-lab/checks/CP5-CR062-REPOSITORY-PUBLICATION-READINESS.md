# CP5-CR062 Repository Publication Readiness 自动检查

## Entry Criteria

| 条件 | 结果 | 说明 |
|---|---|---|
| CR062 正式 CR 已创建 | PASS | CR frontmatter 为 `active-cp5-review-pending`。 |
| CP2 / CP3 gate docs 已创建 | PASS | CP2 / CP3 自动预检和人工审查稿存在。 |
| publication manifest 已冻结 | PASS | `execute_allowed_now=false`，CP5 前不执行 Git 写动作。 |
| scan plan 已创建 | PASS | secret/data/large/path scan 使用 fail-closed。 |
| rollback strategy 已记录 | PASS_WITH_RISK | post-approval failure 不使用 history rewrite / force push。 |

## Checklist

| 检查项 | 结果 | 说明 |
|---|---|---|
| 执行范围最小且明确 | PASS | 仅 repository publication。 |
| post-approval 授权边界明确 | PASS | CP5 DQ 显式列出允许和禁止动作。 |
| 决策项完整 | PASS | CP5 checkpoint 收集 5 项 DQ。 |
| 不授权项完整 | PASS | tag / branch rename / rewrite / NAS / lake / runtime / credentials / relayout 均禁止。 |
| 回滚策略存在 | PASS_WITH_RISK | remote / branch 冲突需停止，不能静默修正。 |
| 用户授权 | PENDING | 等待用户审查。 |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户回复 `approve` 后，才允许 post-approval preflight / scan / manifest / commit / remote / push | PENDING |
| 用户回复 `修改: <具体修改点>` 后返工 | PENDING |
| 用户回复 `reject` 后停止 CR062 | PENDING |

## Deliverables

- `docs/release/REPOSITORY-PUBLICATION-MANIFEST-CR062.yaml`
- `docs/release/GIT-PUBLICATION-SCOPE-CR062.md`
- `docs/release/PUBLICATION-SCAN-PLAN-CR062.md`
- `docs/release/ROLLBACK-REF-CR062.md`
- `process/context/CP5-CR062-REPOSITORY-PUBLICATION-CONTEXT.yaml`
- `process/checkpoints/CP5-CR062-REPOSITORY-PUBLICATION-READINESS.md`
