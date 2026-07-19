---
handoff_id: H-CR051-ST-AW-003-CP7-META-QA-CRITICAL
from_role: host-orchestrator
to_role: meta-qa
canonical_role: meta-qa
workflow_id: meta-flow-self-dev
cr_id: CR-051
story_id: ST-AW-003
stage: verification-execution
status: completed-needs-rework
completed_at: 2026-07-18T14:34:53Z
return_ref: process/returns/ST-AW-003.CP7.return.json
evidence_ref: process/evidence/ST-AW-003.CP7.index.json
created_at: 2026-07-18T14:16:00Z
context_ref: process/context/stories/STORY-ST-AW-003.CP7.verify-packet.json
story_packet_ref: process/context/stories/STORY-ST-AW-003.CP7.verify-packet.json
dispatch: {required: true, mode: subagent, canonical_role: meta-qa, codex_agent_name: qa-cao, reasoning_profile: critical, dispatch_trigger: CP7-heterogeneous-leg-final-verification, tool_name: spawn_agent, agent_id: /root/cr051_staw003_qa, thread_id: /root/cr051_staw003_qa, spawned_at: 2026-07-18T14:17:00Z}
question_permission: {can_ask_user: false, mode: relay-via-host-orchestrator, broker_agent: host-orchestrator}
context_policy:
  capsule_first: true
  story_packet_ref: process/context/stories/STORY-ST-AW-003.CP7.verify-packet.json
  read_profile: minimal
  must_read: [delivery/agents/meta-qa.md, process/context/stories/STORY-ST-AW-003.CP7.verify-packet.json, process/returns/ST-AW-003.CP6.return.json, process/evidence/ST-AW-003.CP6.index.json, process/docs/features/cr051-legs/TEST-PLAN.md]
  read_if_needed: [process/stories/STORY-ST-AW-003-heterogeneous-git-legs-IMPLEMENTATION.md（先写 deep_review read-log）]
  do_not_read_by_default: [process/STATE.md, process/DEVELOPMENT-PLAN.yaml, 其他 Story/LLD/IMPLEMENTATION]
---

# ST-AW-003 CP7 独立验证

独立证明 exact target、typed authz、fresh proof、immutable publication、resume/abort 与 no-cross-leg-rollback。只验证，不改源码/测试；所有 Git/remote 仅临时 fixture。生成四份质量报告和 CP7 return/evidence，不自行写 CP7 result。
