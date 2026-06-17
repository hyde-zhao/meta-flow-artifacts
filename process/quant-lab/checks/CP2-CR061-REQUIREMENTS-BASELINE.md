# CP2-CR061 Requirements Baseline 自动检查

## Entry Criteria

| 条件 | 结果 | 说明 |
|---|---|---|
| CR060 已关闭 | PASS | `process/release/RELEASE-CONTEXT-CR060.yaml` 显示 `closed-current-delivery`。 |
| 用户明确启动 CR061 | PASS | 用户要求继续 quant-lab 迁移并启动 CR061 package/import/layout convergence。 |
| CR046 不恢复 | PASS | 当前请求明确“不要恢复 CR046”。 |
| 恢复上下文可读 | PASS | `process/context/POST-CR060-MIGRATION-HANDOFF-CONTEXT.yaml` 可读。 |

## Checklist

| 检查项 | 结果 | 说明 |
|---|---|---|
| 正式 CR061 可创建 | PASS | 来源 CR060 deferred package/import boundary。 |
| 范围限定 | PASS | 当前只创建门禁和 candidate list，不执行真实代码改动。 |
| 不授权项列明 | PASS | Git remote、NAS、lake、runtime、凭据、CR046 recovery 均禁止。 |
| 冲突预检 | PASS_WITH_RISK | `trading/**` 与 CR046 语义有潜在重叠，当前仅 static candidate。 |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| CR061 baseline 可进入 CP3 / CP5 门禁设计 | PASS |

## Deliverables

- `process/changes/CR-061-PACKAGE-IMPORT-LAYOUT-CONVERGENCE-2026-06-15.md`
- `process/context/CP2-CR061-REQUIREMENT-CONTEXT.yaml`
- `process/checkpoints/CP2-CR061-REQUIREMENTS-BASELINE.md`
