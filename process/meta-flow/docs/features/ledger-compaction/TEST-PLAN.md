---
status: draft
version: "1.0"
feature_id: "FEAT-PG-002"
---

# Ledger Compaction Test Plan

## 测试范围

| Scope ID | 覆盖内容 | 来源 Story / Scenario | 测试层级 | 自动化状态 |
|---|---|---|---|---|
| TP-LC-01 | `meta-flow ledger compact` 与 `state compact` 职责分离 | CR037-S04 / TC-PG-017 | CLI | planned |
| TP-LC-02 | retention policy 解析和非法字段拒绝 | CR037-S04 | unit | planned |
| TP-LC-03 | dry-run plan 不写 ledger | CR037-S04 | integration | planned |
| TP-LC-04 | apply 生成 archive summary/index 和 compacted ledger | CR037-S04 | integration | planned |
| TP-LC-05 | compacted ledger 仍可 event check | CR037-S04 | integration | planned |
| TP-LC-06 | write failure / hash mismatch 回滚 | CR037-S04 | integration | planned |

## 风险驱动测试

| Risk ID | 风险 | 测试方式 | 证据 | 未覆盖原因 |
|---|---|---|---|---|
| R-LC-01 | 原始事件不可恢复 | 对比 archive index、hash 和 compacted marker | pytest tmp fixture | N/A |
| R-LC-02 | 用户误用 `state compact` | help 文案和 CLI negative test | CLI output | N/A |
| R-LC-03 | legacy ledger 行格式混用 | legacy fixture dry-run WARN，不静默 apply | pytest | N/A |

## 权限 / 安全 / 失败路径

| Case ID | 触发条件 | 期望行为 | 测试入口 |
|---|---|---|---|
| SEC-LC-TC-01 | 未传 `--apply` | 只输出 plan，不修改 ledger | CLI test |
| SEC-LC-TC-02 | archive path 写入失败 | abort，原 ledger 不变 | tmp fs fixture |
| SEC-LC-TC-03 | ledger path 指向未授权项目范围 | 拒绝或要求显式授权 | path guard |

## 手工验收

| Check ID | 操作 | 期望结果 | 责任方 |
|---|---|---|---|
| MAN-LC-01 | 审查 compact help | 明确 `state compact` 是 state render/check，`ledger compact` 才是账本压缩 | human / qa |
| MAN-LC-02 | 审查 archive summary | summary 只保存统计、范围、hash、引用，不复制完整长日志 | human / qa |
