---
status: ready-for-review
version: "1.0"
feature_id: "FEAT-WT-05"
---

# Operator and Status Convergence Test Plan

## 测试范围

| Scope ID | 覆盖内容 | 来源 | 层级 | 状态 |
|---|---|---|---|---|
| TP-O-01 | Codex/Claude/Qoder 非交互 full dry-run 3/3 | ST-WT-006 / TC-WT-006 | dry-run | planned |
| TP-O-02 | README command 与 CLI contract 一致 | REQ-WT-013 | docs contract | planned |
| TP-O-03 | CR-046 7/7 status + stale statement=0 | ST-WT-007 / TC-WT-007 | evidence integration | planned |
| TP-O-04 | object identity ambiguity/prefix collision/hash mutation | CP3-DQ-05 | negative fixture | planned |
| TP-O-05 | additive CP7/CP8 ceiling | CP3-DQ-06 | verdict fixture | planned |

## 风险驱动与失败路径

| Case | 触发 | 期望 | 入口 |
|---|---|---|---|
| O-NEG-01 | 非 TTY 且缺 `--project-dir` | 明确失败；文档示例不得缺参 | installer CLI test |
| O-NEG-02 | 同目录 CR-046/047 prefix collision | 按 formal identity 分离 | `tests/test_cr047_operator_status.py` |
| O-NEG-03 | protected original hash 变化/身份不明 | BLOCKED + child-CR route | 同上 |
| O-NEG-04 | fixture 冒充 receipt/independent QA | 拒绝声明并保持 risk OPEN | 同上 |

## 手工验收

| Check | 操作 | 期望 | 责任方 |
|---|---|---|---|
| MAN-O-01 | 审查三平台文档命令 | 均显式 project scope/full/project-dir/dry-run | CP7 reviewer |
| MAN-O-02 | 对照 manifest before/after | protected hash changes=0 | CP6/CP7 reviewer |
