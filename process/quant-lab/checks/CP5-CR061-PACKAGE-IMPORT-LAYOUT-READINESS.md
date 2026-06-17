# CP5-CR061 Package / Import / Layout Readiness 自动检查

## Entry Criteria

| 条件 | 结果 | 说明 |
|---|---|---|
| CR061 正式 CR 已创建 | PASS | CR frontmatter 为 `active-cp5-review-pending`。 |
| candidate list 已冻结 | PASS | CR061-CAND-001..016 已列出。 |
| preserve-audit decisions 已创建 | PASS | 历史证据默认 preserve-audit。 |
| rollback refs 已记录 | PASS_WITH_RISK | CR059 bundle present by reference；真实实现前需刷新或 waiver。 |
| execution manifest 已创建 | PASS | `execute_allowed_now=false`。 |

## Checklist

| 检查项 | 结果 | 说明 |
|---|---|---|
| 执行范围最小且明确 | PASS | 当前 gate-only。 |
| 决策项完整 | PASS | CP5 checkpoint 收集 5 项 DQ。 |
| 不授权项完整 | PASS | Git/NAS/lake/runtime/credentials/CR046 recovery 均禁止。 |
| 回滚策略存在 | PASS_WITH_RISK | dirty worktree include/exclude 未冻结，阻断真实实现。 |
| 用户授权 | PENDING | 等待用户审查。 |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 用户回复 `approve` 后只批准 CR061 设计门禁，不执行真实迁移 | PENDING |
| 用户回复 `修改: <具体修改点>` 后返工 | PENDING |
| 用户回复 `reject` 后停止 CR061 | PENDING |

## Deliverables

- `docs/release/MIGRATION-EXECUTION-MANIFEST-CR061.yaml`
- `docs/release/CR061-PACKAGE-IMPORT-CANDIDATE-LIST.md`
- `docs/release/REPO-LOCAL-PACKAGE-LAYOUT-PLAN-CR061.md`
- `docs/release/PRESERVE-AUDIT-DECISIONS-CR061.md`
- `docs/release/ROLLBACK-REF-CR061.md`
- `process/context/CP5-CR061-LLD-CONTEXT.yaml`
- `process/checkpoints/CP5-CR061-PACKAGE-IMPORT-LAYOUT-READINESS.md`
