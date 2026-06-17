# CP3-CR062 HLD / Architecture 自动检查

## Entry Criteria

| 条件 | 结果 | 说明 |
|---|---|---|
| CR062 baseline 已创建 | PASS | 正式 CR 文件已创建。 |
| publication scope 已创建 | PASS | `docs/release/GIT-PUBLICATION-SCOPE-CR062.md`。 |
| manifest / scan / rollback 已创建 | PASS | CR062 release gate 文档已生成。 |

## Checklist

| 检查项 | 结果 | 说明 |
|---|---|---|
| gate-first 架构 | PASS | CP2/CP3/CP5 approve 前不执行 Git 写动作。 |
| fail-closed scan 架构 | PASS | secret/data/large/path scan 必须在 post-approval preflight PASS。 |
| branch policy | PASS_WITH_RISK | 不 rename branch；若目标 branch 策略冲突，停止并问用户。 |
| process evidence publication policy | PASS_WITH_RISK | 推荐 selective evidence，不发布全量历史过程文件。 |
| remote conflict policy | PASS | 现有 remote 冲突时停止，不静默 set-url。 |
| external operation boundary | PASS | NAS/lake/provider/runtime/凭据/CR046 recovery 均排除。 |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 可进入 CP5 repository publication readiness gate | PASS |

## Deliverables

- `docs/release/GIT-PUBLICATION-SCOPE-CR062.md`
- `docs/release/REPOSITORY-PUBLICATION-MANIFEST-CR062.yaml`
- `docs/release/PUBLICATION-SCAN-PLAN-CR062.md`
- `docs/release/ROLLBACK-REF-CR062.md`
- `process/context/CP3-CR062-DESIGN-CONTEXT.yaml`
- `process/checkpoints/CP3-CR062-HLD-REVIEW.md`
