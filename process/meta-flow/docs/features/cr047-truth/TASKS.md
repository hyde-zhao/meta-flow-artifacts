---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-01"
---

# Truth Consistency Tasks

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | Owner | 验证 |
|---|---:|---|---|---|---|---|
| TASK-WT-001-01 | 1 | 扩展 lifecycle/State/CR/CURRENT relation checker | ADR-WT-001 | `meta_flow/checks/cr_tracking.py`, `meta_flow/state/current.py` | ST-WT-001 primary | TP-T-01/02 |
| TASK-WT-001-02 | 2 | 迁移 JSON-only canonical tracking 并登记 CR-033 candidate | REQ-WT-003 | `process/changes/CR-INDEX.json`, legacy index migration | ST-WT-001 primary | TP-T-03 |
| TASK-WT-001-03 | 3 | 增加 stale/current/status-sync 回归 fixtures | 前两项 | `tests/test_cr047_truth_consistency.py` + existing tests | ST-WT-001 primary | TP-T-01..04 |

## 阻塞项

无；若现有 schema 无法表达批准对象，停止并重开架构决策。
