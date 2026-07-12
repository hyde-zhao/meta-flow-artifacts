---
status: draft
version: "1.0"
feature_id: "FEAT-EI-GOVERNANCE"
---

# Feature Test Plan: Workspace and Lifecycle Governance

| Case | 触发 | 期望 |
|---|---|---|
| CT-GOV-01 | valid symlink/local metadata | shared resolver一致 PASS/compat warning |
| CT-GOV-02 | non-null dangling/conflicting metadata | workspace/state/doctor/CP 全部 FAIL/BLOCKED |
| CT-GOV-03 | phase-in-progress vs gate-open | 正确状态接受；future gate/checklist缺失拒绝 |
| CT-GOV-04 | delivered有active refs/错误health owner | reject |
| CT-GOV-05 | 未授权全文扩展 | reject并给 safe route |
| CT-GOV-06 | compact/restore typed digest相同 | apply allowed |
| CT-GOV-07 | node/edge/terminal/correction/health任一变化 | no apply；源hash不变 |
| CT-GOV-08 | metadata migration candidate失败 | 不写正式ref；可回 explicit legacy-null |

要求：dangling被PASS次数=0；semantic mismatch apply次数=0；原历史mutation=0。回归覆盖 workspace routing、state transition、context pack和ledger compaction。
