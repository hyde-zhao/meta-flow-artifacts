---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-03"
---

# Quality Governance Test Plan

## 测试范围

| Scope ID | 覆盖内容 | 来源 | 层级 | 状态 |
|---|---|---|---|---|
| TP-Q-01 | active/default-required/cold/unknown 分类 | ST-WT-003 / TC-WT-003 | unit | planned |
| TP-Q-02 | B0_pre/B0_cp7 与 explainable delta | REQ-WT-006 | contract | planned |
| TP-Q-03 | Quality Model derived source 与 6 个历史 correction | REQ-WT-007 | integration | planned |
| TP-Q-04 | blocker/warning exit semantics | REQ-WT-008 | doctor | planned |
| TP-Q-05 | future Run event / legacy-unverified rejection | REQ-WT-017 | ledger | planned |

## 风险驱动与失败路径

| Case | 触发 | 期望 | 入口 |
|---|---|---|---|
| Q-NEG-01 | active over-budget 或 unclassified | Doctor 非零 | `tests/test_cr047_quality_governance.py` |
| Q-NEG-02 | delta 无 reason/remediation | 非零 | 同上 |
| Q-NEG-03 | 伪造历史时间/receipt/actor | 100% 拒绝 | 同上 |
| Q-POS-01 | 仅 classified cold warning | exit=0 且 warning count>0 | Doctor fixture |

## 手工验收

| Check | 操作 | 期望 | 责任方 |
|---|---|---|---|
| MAN-Q-01 | 对照 B0_cp7 report | 每个新增/删除/重分类有解释 | CP7 reviewer |
