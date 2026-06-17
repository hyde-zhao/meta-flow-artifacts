# CP3-CR061 HLD / Architecture 自动检查

## Entry Criteria

| 条件 | 结果 | 说明 |
|---|---|---|
| CR061 baseline 已创建 | PASS | 正式 CR 文件已创建。 |
| candidate list 已创建 | PASS | `docs/release/CR061-PACKAGE-IMPORT-CANDIDATE-LIST.md`。 |
| layout plan 已创建 | PASS | `docs/release/REPO-LOCAL-PACKAGE-LAYOUT-PLAN-CR061.md`。 |

## Checklist

| 检查项 | 结果 | 说明 |
|---|---|---|
| staged architecture | PASS | 推荐 staged compatibility-first。 |
| legacy import compatibility | PASS | `engine` / `market_data` / `strategies` / `trading` 默认保留。 |
| bulk move boundary | PASS_WITH_RISK | Bulk move 延后到二次授权。 |
| runtime / credential boundary | PASS | 不读取 `.env`，不启动 provider/QMT/MiniQMT。 |
| CR046 overlap control | PASS_WITH_RISK | `trading/**` 只允许 static import/layout review。 |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 可进入 CP5 package/import/layout readiness gate | PASS |

## Deliverables

- `docs/release/REPO-LOCAL-PACKAGE-LAYOUT-PLAN-CR061.md`
- `docs/release/PRESERVE-AUDIT-DECISIONS-CR061.md`
- `process/context/CP3-CR061-DESIGN-CONTEXT.yaml`
- `process/checkpoints/CP3-CR061-HLD-REVIEW.md`
