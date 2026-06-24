---
status: "ready_with_risk"
release_decision: "READY_WITH_RISK"
release_artifact_profile: "compact"
change_id: "CR-138"
created_at: "2026-06-24T18:58:00+08:00"
---

# CR138 Release Notes

## 结论

CR138 当前交付关闭为 `READY_WITH_RISK`。本次关闭范围是 Runner Control Plane 与 QMT Gateway Service Layer 的本地合同、fixture、文档、guardrail、no-real-operation counters 和相关回归基线。

本次关闭不表示真实 QMT / MiniQMT / XtQuant / gateway runtime、凭据、账户、行情、订单、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 已授权或已验证。

## 用户可见变化

| 项 | 内容 |
|---|---|
| Runner baseline | 增加 run plan、preflight、command decision、event、rebalance draft、ops summary、evidence/review/incident 合同。 |
| Gateway baseline | 增加 REST-only route contract、health/capabilities/session/change plan、calendar/commission/PnL、subscription/order report/recovery 合同。 |
| Authorization | 增加 CR138 authorization runbook，明确按需授权和真实操作禁止项。 |
| Fixture matrix | 增加 `process/docs/quality/TEST-MATRIX-CR138.md`，记录 S01..S08 与 no-real-operation counters。 |

## 质量摘要

| 验证项 | 结果 |
|---|---|
| CR138 fixture / docs / hygiene tests | PASS，48 passed |
| QMT / runner targeted regression | PASS，35 passed |
| Artifact hygiene | PASS，unclassified=0 |
| State check | PASS |
| py_compile | PASS |
| git diff --check | PASS |

## 已知风险

| 风险 | 状态 | 说明 |
|---|---|---|
| RISK-CR138-RUNTIME-NOT-VERIFIED | accepted-by-scope | 真实 runtime、账户、行情、订单和交易动作未验证。 |
| RISK-CR138-RUNTIME-MISREAD | controlled | 当前交付可能被误读为 runtime-ready；所有 release/CP8 文档继续声明 not-authorized。 |

## 后续候选

| 候选 | 说明 |
|---|---|
| CR138-FU-RT-01 | Gateway xtquant adapter Spike / readonly integration design。 |
| CR138-FU-RT-02 | QMT terminal direct-run validation authorization gate。 |
| CR138-FU-TRD-01 | Order-write / simulation / live design authorization gate。 |
| CR138-FU-GOV-01 | CR tracking warning cleanup。 |
