# CP2-CR062 Requirements Baseline 自动检查

## Entry Criteria

| 条件 | 结果 | 说明 |
|---|---|---|
| CR061 已关闭 | PASS | CR061 Batch B 为 `closed-current-delivery / READY_WITH_RISK`。 |
| 用户明确启动 CR062 | PASS | 用户要求按 handoff 建议启动 Git Remote Cutover / Repository Publication Gate。 |
| CR046 不恢复 | PASS | 当前请求明确“不要恢复 CR046”。 |
| 禁止操作边界明确 | PASS | CP gate 批准前不执行 remote / push / tag / branch rename / history rewrite、NAS/lake/provider/runtime/凭据、physical move 或 bulk import rewrite。 |

## Checklist

| 检查项 | 结果 | 说明 |
|---|---|---|
| 正式 CR062 可创建 | PASS | 来源为 CR061 CP8 DQ-CP8-CR061-03。 |
| 范围限定 | PASS | 当前只创建门禁、scope、manifest、scan plan 和 rollback。 |
| GitHub target candidate 明确 | PASS | `git@github.com:hyde-zhao/quant-lab.git` 仅作为候选，不写入 Git config。 |
| 不授权项列明 | PASS | Git 写动作、NAS/lake/runtime/凭据、physical move/bulk rewrite 均禁止。 |
| 冲突预检 | PASS_WITH_RISK | CR046 仍 active-paused；CR062 不触发 runtime 或交易。 |

## Exit Criteria

| 条件 | 结果 |
|---|---|
| CR062 baseline 可进入 CP3 / CP5 publication gate | PASS |

## Deliverables

- `process/changes/CR-062-GIT-REMOTE-CUTOVER-REPOSITORY-PUBLICATION-GATE-2026-06-15.md`
- `process/context/CP2-CR062-REQUIREMENT-CONTEXT.yaml`
- `process/checkpoints/CP2-CR062-REQUIREMENTS-BASELINE.md`
