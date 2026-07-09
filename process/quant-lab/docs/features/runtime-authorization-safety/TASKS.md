---
status: "draft-current-index"
version: "1.0"
feature_id: "FEAT-07"
---

# Feature Tasks: runtime-authorization-safety

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.1 | 2026-07-05 | host-orchestrator | CR158 增补 adapter forbidden counters、blocked reason 和 release overclaim guard 任务。 |

| Task ID | 任务 | 输入 | 输出 | 文件范围 | 验证 |
|---|---|---|---|---|---|
| FEAT-07-T01 | 维护授权语义表 | CP / CR / runbook | authorization meaning matrix | docs / process checks | docs guardrail |
| FEAT-07-T02 | 维护 no-real-operation counters | high-risk CR | safety counters | tests / reports / docs | no-real-op tests |
| FEAT-07-T03 | 维护凭据脱敏策略 | `.env.example` / credential_ref | redaction policy | `trading/qmt_redaction.py`、docs | redaction tests |
| FEAT-07-T04 | 维护 HMAC / scope / nonce guardrail | gateway scope | fail-closed auth | `trading/qmt_auth.py` | auth tests |
| FEAT-07-T05 | 维护禁止依赖扫描 | dependency map | forbidden import checks | tests / scripts | boundary tests |
| FEAT-07-T06 | CR158 adapter forbidden counter report | CR158 HLD/ADR、REQ-CR158-005 | AdapterOperationCounterReport、blocked reason refs、CP7 evidence hooks | planned adapter safety module / tests / docs | planned `tests/research/test_strategy_adapter_no_runtime_guard.py` |
| FEAT-07-T07 | CR158 release overclaim guard | CR158 CP8 wording boundary | no-runtime/no-registry/no-publish wording checklist | docs/release and component docs | CP8 review |

## 后续触发条件

- 用户恢复 CR-020 真实只读验证。
- 启动 CR-021..024。
- 新增 provider、lake write、publish、gateway endpoint、account query、simulation/live 能力。
- CR158 或后续 CR 申请真实 event feed、真实 ML training、model registry、provider/lake/NAS/credential、runtime、trading、publish 或 Git remote write 时。
