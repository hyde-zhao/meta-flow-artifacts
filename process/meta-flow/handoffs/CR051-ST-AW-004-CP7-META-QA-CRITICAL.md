---
handoff_id: H-CR051-ST-AW-004-CP7-META-QA-CRITICAL
from_role: host-orchestrator
to_role: meta-qa
canonical_role: meta-qa
workflow_id: meta-flow-self-dev
cr_id: CR-051
story_id: ST-AW-004
stage: verification-execution
status: completed
completed_at: 2026-07-18T14:33:23Z
return_ref: process/returns/ST-AW-004.CP7.return.json
evidence_ref: process/evidence/ST-AW-004.CP7.index.json
created_at: 2026-07-18T14:16:00Z
context_ref: process/context/stories/STORY-ST-AW-004.CP7.verify-packet.json
story_packet_ref: process/context/stories/STORY-ST-AW-004.CP7.verify-packet.json
dispatch: {required: true, mode: subagent, canonical_role: meta-qa, codex_agent_name: qa-yan, reasoning_profile: critical, dispatch_trigger: CP7-aggregate-evidence-final-verification, tool_name: spawn_agent, agent_id: /root/cr051_staw004_qa, thread_id: /root/cr051_staw004_qa, spawned_at: 2026-07-18T14:17:00Z}
question_permission: {can_ask_user: false, mode: relay-via-host-orchestrator, broker_agent: host-orchestrator}
context_policy:
  capsule_first: true
  story_packet_ref: process/context/stories/STORY-ST-AW-004.CP7.verify-packet.json
  read_profile: minimal
  must_read: [delivery/agents/meta-qa.md, process/context/stories/STORY-ST-AW-004.CP7.verify-packet.json, process/returns/ST-AW-004.CP6.return.json, process/evidence/ST-AW-004.CP6.index.json, process/docs/features/cr051-aggregate/TEST-PLAN.md]
  read_if_needed: [process/stories/STORY-ST-AW-004-aggregate-evidence-gate-IMPLEMENTATION.md（先写 deep_review read-log）]
  do_not_read_by_default: [process/STATE.md, process/DEVELOPMENT-PLAN.yaml, 其他 Story/LLD/IMPLEMENTATION]
---

# ST-AW-004 CP7 独立验证

独立证明 16 组合、published-handle fail-closed、top-level-only digest omit、immutable store、selector CAS 和 persisted 2/2 PASS 投影硬门。只验证，不改源码/测试；不得调用 Git/worktree/close/status-sync/rollback。生成四份质量报告和 CP7 return/evidence，不自行写 CP7 result。
