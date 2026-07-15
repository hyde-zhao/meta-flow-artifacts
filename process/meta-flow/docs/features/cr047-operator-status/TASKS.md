---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-05"
---

# Operator and Status Convergence Tasks

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | Owner | 验证 |
|---|---:|---|---|---|---|---|
| TASK-WT-007-01 | 1 | 在 CP6 pre-implementation 生成对象身份制 protected manifest | CP3-DQ-05 | `process/evidence/CR047-ST-WT-007.protected-originals.json` | ST-WT-007 primary | TP-O-04 |
| TASK-WT-007-02 | 2 | 收敛 CR-046 current product/status projection | formal 7/7 evidence | product matrix/current refs/new correction only | ST-WT-007 primary | TP-O-03/05 |
| TASK-WT-007-03 | 3 | CP6 首验并为 CP7 保留再验入口 | manifest | hash check result/evidence index | ST-WT-007 primary | TP-O-04 |
| TASK-WT-006-01 | 4 | 同步三平台非交互 dry-run 文档入口 | existing installer contract | `README.md`, `delivery/README.md`, `delivery/doc/USER-MANUAL.md` | ST-WT-006 primary | TP-O-01/02 |
| TASK-WT-006-02 | 5 | 把 cache preflight 与 3 平台 dry-run 纳入操作说明/验证 | FEAT-WT-04 contract | docs + installer/preflight tests | ST-WT-006 primary/shared | TP-O-01/02 |

## 阻塞项

protected identity/hash 不明或原件变化时立即停止 ST-WT-007 并拆子 CR；不得继续 TASK-WT-007-02。
