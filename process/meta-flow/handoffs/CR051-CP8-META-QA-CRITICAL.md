---
handoff_id: "HO-CR051-CP8-META-QA-CRITICAL"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "release-readiness"
from_role: "host-orchestrator"
to_role: "meta-qa"
codex_agent_name: "meta-qa-critical"
reasoning_profile: "critical"
status: "completed-with-risk"
context_ref: "process/context/CP8-CR051-RELEASE-READINESS-CONTEXT.yaml"
created_at: "2026-07-18T16:50:00Z"
dispatch:
  canonical_role: "meta-qa"
  codex_agent_name: "meta-qa-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "documentation-complete-all-five-cp7-complete"
  tool_name: "spawn_agent"
  agent_id: "/root/cr051_cp8_meta_qa"
  thread_id: "/root/cr051_cp8_meta_qa"
completed_at: "2026-07-18T17:04:00Z"
return_ref: "process/returns/CR051.CP8-PREP.return.json"
evidence_ref: "process/evidence/CR051.CP8-PREP.index.json"
release_context_ref: "process/release/RELEASE-CONTEXT-CR051.yaml"
---

# CR-051 CP8 Release Readiness Handoff

当且仅当 `process/returns/CR051-DOCUMENTATION.return.json` 已形成后，按 capsule-first 执行 CR-051 聚合质量复核和 release-readiness。输出 CR 级 Verification/Test/Review/Fixes 摘要、full-profile 五份 release 文档、release context、return 与 evidence index。结论最高为 `READY_WITH_RISK`，不得写成 `RELEASED`。

必须披露全部五个 Story 的最新 CP7、两个已关闭 HIGH finding、ST-AW-001/ST-AW-002 的 bounded Host fallback 独立性上限、ST-AW-005 首轮 QA finding 与证据写入 fallback 的真实 provenance，以及所有 not-authorized 边界。不得修改实现、测试、用户文档、State、ledger、checkpoint、CP result 或 CR；不得执行真实 Git/worktree/ref/remote/link/migration/publish 操作。
