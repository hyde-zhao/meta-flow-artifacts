# CP5-CR061 Batch B Implementation Authorization 自动检查

## Entry Criteria

| 条件 | 结果 | 证据 |
|---|---|---|
| CR061 gate-only CP5 已 approved | PASS | `process/checkpoints/CP5-CR061-PACKAGE-IMPORT-LAYOUT-READINESS.md` |
| Batch B dirty worktree include/exclude 已冻结 | PASS | `docs/release/CR061-BATCH-B-DIRTY-WORKTREE-INCLUDE-EXCLUDE.yaml` |
| Batch B candidate manifest 已冻结 | PASS | `docs/release/CR061-BATCH-B-CANDIDATE-MANIFEST.yaml` |
| Feature design / test plan / tasks 已生成 | PASS | `docs/features/cr061-package-import-layout/` |
| 不授权边界已保留 | PASS | 本文件与 checkpoint 不授权项 |

## Checklist

| 检查项 | 结果 | 说明 |
|---|---|---|
| 当前是否仍未执行代码改动 | PASS | 未修改 `pyproject.toml`、`uv.lock`、源码或 import。 |
| Batch B 范围是否最小 | PASS | 限定 metadata、lock、可选 `quant_lab` namespace、smoke test。 |
| dirty worktree 是否显式 include/exclude | PASS | 非 CR061 Batch B 文件默认排除。 |
| rollback_ref 是否可作为执行前阻断项 | PASS_WITH_RISK | CR059 bundle 当前仅 planning-only；实现前需刷新或风险接受。 |
| Git/NAS/lake/runtime/credential 是否禁止 | PASS | 均列入 blocked actions。 |
| CR046 是否保持暂停 | PASS | 不恢复、不推进 CP7。 |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| 可发起人工门禁 | PASS |
| 人工门禁批准前不得实现 | PASS |
| 批准后也只允许 Batch B-ACT-001..005 | PASS |

## Deliverables

- `docs/release/CR061-BATCH-B-DIRTY-WORKTREE-INCLUDE-EXCLUDE.yaml`
- `docs/release/CR061-BATCH-B-CANDIDATE-MANIFEST.yaml`
- `docs/design/FEATURE-DESIGN-MATRIX-CR061.md`
- `docs/features/cr061-package-import-layout/DESIGN.md`
- `docs/features/cr061-package-import-layout/TEST-PLAN.md`
- `docs/features/cr061-package-import-layout/TASKS.md`
- `process/context/CP5-CR061-BATCH-B-AUTHORIZATION-CONTEXT.yaml`
- `process/checkpoints/CP5-CR061-BATCH-B-IMPLEMENTATION-AUTHORIZATION.md`

## 结论

自动预检结论：PASS_WITH_RISK。

风险来自 rollback_ref 当前仍复用 CR059 planning-only bundle，以及 dirty worktree 中存在大量历史迁移过程文件。该风险可通过 CP5 Batch B 人工门禁中的 DQ-CP5-CR061B-02 / DQ-CP5-CR061B-03 接受或要求返工。
