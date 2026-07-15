---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-01"
---

# Truth Consistency Test Plan

## 测试范围

| Scope ID | 覆盖内容 | 来源 | 层级 | 状态 |
|---|---|---|---|---|
| TP-T-01 | closed/cancelled/superseded/missing/candidate/active lifecycle 表格 | ST-WT-001 / TC-WT-001 | unit | planned |
| TP-T-02 | State/CR/CURRENT 五类 scalar/ref drift | REQ-WT-001..002 | contract | planned |
| TP-T-03 | JSON-only index、legacy YAML 与 CR-033 candidate | REQ-WT-003 | integration | planned |
| TP-T-04 | status-sync→refresh→relation check | ADR-WT-001 | regression | planned |

## 风险驱动与失败路径

| Case | 触发 | 期望 | 入口 |
|---|---|---|---|
| T-NEG-01 | active_change=closed CR-037 | 非零并定位 lifecycle conflict | `tests/test_cr047_truth_consistency.py` |
| T-NEG-02 | CURRENT context/checkpoint 与 State 不同 | 非零并列出字段 | 同上 |
| T-NEG-03 | YAML 并存或 candidate 缺失 | cr-tracking 非零 | `tests/test_gate_policy.py` |
| T-POS-01 | CR-047 active、JSON/index/current 一致 | blocker=0 | `meta-flow check cr-tracking` + state check |

## 手工验收

| Check | 操作 | 期望 | 责任方 |
|---|---|---|---|
| MAN-T-01 | 检查错误消息 | 包含 source、field、expected/actual、remediation | CP7 reviewer |
