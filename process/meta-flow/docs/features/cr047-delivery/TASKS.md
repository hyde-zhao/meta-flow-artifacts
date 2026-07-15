---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-04"
---

# Deterministic Delivery Gate Tasks

| TASK-ID | 顺序 | 任务 | 输入 | 输出文件 | Owner | 验证 |
|---|---:|---|---|---|---|---|
| TASK-WT-004-01 | 1 | 将 tracked delivery rule 设为 guardrail canonical | ADR-WT-004 | `scripts/check_delivery_guardrails.py`, `delivery/rules/AGENTS.md`, `.gitignore` | ST-WT-004 primary | TP-D-01 |
| TASK-WT-004-02 | 2 | 实现 tracked/package/ignored cache classifier | REQ-WT-010/014 | guardrail + fixtures | ST-WT-004 primary | TP-D-02 |
| TASK-WT-005-01 | 3 | 分批修复 Ruff 90 项并人工处理 B/F | REQ-WT-011 | affected Python files/tests | ST-WT-005 merge owner | TP-D-03 |
| TASK-WT-005-02 | 4 | 把 Ruff/pytest 纳入现有组合 preflight | ADR-WT-004 | guardrail/preflight integration | ST-WT-005 primary/shared | TP-D-05 |
| TASK-WT-005-03 | 5 | 执行完整无缓存回归并固化计数 | REQ-WT-012 | Run event + test evidence | ST-WT-005 primary | TP-D-04 |

## 阻塞项

ST-WT-005 对 ST-WT-003/004 共享 Python/guardrail 文件使用 file-conflict 依赖，必须在其实现合并后执行 Ruff 批处理。
