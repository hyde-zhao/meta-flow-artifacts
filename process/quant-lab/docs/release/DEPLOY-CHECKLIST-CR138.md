---
status: "ready_with_risk"
release_decision: "READY_WITH_RISK"
release_artifact_profile: "compact"
change_id: "CR-138"
created_at: "2026-06-24T18:58:00+08:00"
---

# CR138 Deploy Checklist

## Scope

本检查清单只覆盖本地代码、测试、文档和过程产物收口。不包含真实部署、真实 gateway 启动、端口绑定、凭据读取、账户/行情/订单查询、submit/cancel、simulation/live、NAS、provider/lake/catalog 或 Git remote 写入。

## Checklist

| 检查项 | 状态 | 证据 |
|---|---|---|
| CP6 implementation PASS | PASS | `process/checks/CP6-CR138-RUNNER-QMT-OPERATIONAL-CONTROL-CODING-DONE.md` |
| CP7 fixture/static verification PASS_WITH_RISK | PASS | `process/checks/CP7-CR138-RUNNER-QMT-OPERATIONAL-CONTROL-VERIFICATION-DONE.md` |
| Release context ready | PASS | `process/release/RELEASE-CONTEXT-CR138.yaml` |
| CR138 fixture tests pass | PASS | `process/evidence/CR138-BATCH.CP7.index.json` |
| Related regression pass | PASS | `process/evidence/CR138-BATCH.CP7.index.json` |
| Artifact hygiene pass | PASS | `unclassified=0` |
| State check pass | PASS | `State v2 Check: OK` |
| No runtime authorization implied | PASS | `runtime_authorization_allowed=false` |

## Not Authorized

| 操作 | 状态 |
|---|---|
| QMT / MiniQMT / XtQuant / gateway runtime | not authorized |
| Credential / account / market / order read | not authorized |
| submit / cancel / simulation / live | not authorized |
| NAS / provider / lake / catalog | not authorized |
| Git remote write | not authorized |
