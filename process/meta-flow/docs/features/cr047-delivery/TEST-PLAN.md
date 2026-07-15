---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-04"
---

# Deterministic Delivery Gate Test Plan

## 测试范围

| Scope ID | 覆盖内容 | 来源 | 层级 | 状态 |
|---|---|---|---|---|
| TP-D-01 | clean archive canonical rule/ignored wrapper | ST-WT-004 / TC-WT-004 | archive integration | planned |
| TP-D-02 | ignored/tracked/package cache 分类 | REQ-WT-010/014 | fixture | planned |
| TP-D-03 | Ruff error=0 | ST-WT-005 / REQ-WT-011 | static | planned |
| TP-D-04 | pytest ≥377 tests +70 subtests | REQ-WT-012 | regression | planned |
| TP-D-05 | 组合门单步失败传播与 Run refs | ADR-WT-004 | contract | planned |

## 风险驱动与失败路径

| Case | 触发 | 期望 | 入口 |
|---|---|---|---|
| D-NEG-01 | tracked canonical rule 缺失 | BLOCKER | `tests/test_cr047_delivery_gate.py` |
| D-NEG-02 | tracked/package cache | BLOCKER | 同上 |
| D-NEG-03 | pure ignored cache | WARN/cleanup，不形成永久失败 | 同上 |
| D-NEG-04 | Ruff/pytest 任一步非零 | 组合门非零且保留 step ref | 同上 |

## 手工验收

| Check | 操作 | 期望 | 责任方 |
|---|---|---|---|
| MAN-D-01 | 审查 Ruff 非自动项 | B/F 语义问题均有人工确认 | CP7 reviewer |
