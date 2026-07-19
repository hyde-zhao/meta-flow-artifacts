---
change_id: CR-051
report_type: test-report
revision: R2
status: PASS_WITH_RISK
date: 2026-07-19
---

# CR-051 CP8 终验回修测试报告

## 结果摘要

| 指标 | 结果 |
|---|---:|
| 全仓 tests | 697 passed / 697 executed |
| 全仓 subtests | 70 passed / 70 executed |
| failures / errors / skipped blockers | 0 / 0 / 0 |
| design delta merged | 3 / 3 |
| LLD structure check | 3 / 3 PASS |
| 真实仓库/remote/worktree mutation | 0 |

## 关键新增覆盖

- ST-AW-002：typed proof 身份/attempt/target/before/expiry，calibration revoke/persistence，owner 跨项目隔离，record identity tamper，非法 phase，final idempotency。
- ST-AW-003：CR-050 compatibility、source/artifact 独立失败、dirty sibling 隔离、finish containment + expected-OID CAS cleanup。
- ST-AW-004：consumer target policy、dependency boundary、exact evidence DAG、state-success/ledger-failure PARTIAL、retry COMPLETE。

## 非阻断说明

全仓 format baseline 在当前 Ruff 版本下存在 92 个历史文件需重排；本次核心变更文件 11/11 scoped format PASS。未获授权的 hosted remote、Windows native 与真实迁移测试未执行，已进入 CP8 风险接受项。
