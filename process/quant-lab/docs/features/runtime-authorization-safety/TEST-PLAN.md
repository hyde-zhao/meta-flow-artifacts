---
status: "draft-current-index"
version: "1.0"
feature_id: "FEAT-07"
---

# Feature Test Plan: runtime-authorization-safety

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-05 | host-orchestrator | CR158 增补 event/ML adapter forbidden operation counter 测试入口。 |

## 测试矩阵

| 测试域 | 关键场景 | 验证入口 |
|---|---|---|
| no-real-operation | 未授权时 provider/lake/publish/QMT/broker/credential 计数为 0 | `tests/test_cr025_no_real_operation_safety.py`、`tests/test_cr030_no_real_operation_safety.py`、CR020 tests |
| HMAC / scope | timestamp、nonce、scope、allowlist fail-closed | `tests/test_cr019_qmt_pairing_hmac_auth.py`、`tests/test_cr020_hmac_pairing_allowlist_scope.py` |
| redaction | 日志 / docs / evidence 不含 token、账户、密码、session、私有路径 | QMT redaction tests / docs tests |
| forbidden dependency | C 侧不导入 xtquant，consumer 不导入 provider/runtime | CR019 / CR020 / CR010 consumer boundary tests |
| docs authorization guardrail | README / runbook 不把 verified 写成真实授权 | `tests/test_cr020_docs_runbook_no_authorization.py`、CR016 docs tests |
| stage / runtime gate | health / capabilities / CP pass 不升级权限 | CR019 run gate tests、CR020 runtime tests |
| CR158 adapter forbidden counters | event feed、model training、model registry、provider/lake/NAS/credential、runtime/trading、publish、Git remote 计数均为 0；任一非 0 blocked | planned `tests/test_cr158_adapter_no_runtime_guard.py`、CP7 evidence scan |
| CR158 release overclaim guard | release wording 不得把 fixture/static adapter readiness 写成 production/runtime/trading/model-registry readiness | planned docs/release review + CP8 checklist |

## 手工验证

| 场景 | 证据要求 |
|---|---|
| 用户提交真实运行 evidence | 必须脱敏；只接受摘要、计数、scope、blocked reason，不接受凭据原文 |
| 恢复 CR-020 | 必须重新确认 MiniQMT 权限、只读 scope 和不授权交易项 |
| CR158 申请真实 feed / training / registry / runtime 验证 | 必须另起 runtime authorization CR；本 CR CP3/CP4/CP5/CP6/CP7/CP8 均不隐含授权 |
