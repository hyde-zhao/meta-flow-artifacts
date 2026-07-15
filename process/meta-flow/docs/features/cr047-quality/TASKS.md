---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-03"
---

# Quality Governance Tasks

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | Owner | 验证 |
|---|---:|---|---|---|---|---|
| TASK-WT-003-01 | 1 | 增加 lifecycle/read classification 与 typed finding | ADR-WT-003 | `meta_flow/checks/token_budget.py`, quality checker | ST-WT-003 primary | TP-Q-01/02/04 |
| TASK-WT-003-02 | 2 | 修正 Quality Model source 并定义 append-only correction | REQ-WT-007 | `process/policies/QUALITY-MODEL.yaml`, ledgers/correction refs | ST-WT-003 primary | TP-Q-03 |
| TASK-WT-003-03 | 3 | 强制 future Run event、历史 legacy-unverified 语义 | REQ-WT-017 | workflow Doctor / Run ledger contract | ST-WT-003 primary | TP-Q-05 |
| TASK-WT-003-04 | 4 | 增加双基线/Doctor/历史 fixtures | 前三项 | `tests/test_cr047_quality_governance.py` + existing tests | ST-WT-003 primary | TP-Q-01..05 |

## 阻塞项

无法分类的 current/default-required 对象保持 blocker；不允许用 warning 规避。
