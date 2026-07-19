---
handoff_id: H-CR051-ST-AW-003-CP7-R2-META-QA-CRITICAL
from_role: host-orchestrator
to_role: meta-qa
canonical_role: meta-qa
workflow_id: meta-flow-self-dev
cr_id: CR-051
story_id: ST-AW-003
stage: verification-reexecution
status: completed-with-risk
completed_at: 2026-07-18T15:20:00Z
return_ref: process/returns/ST-AW-003.CP7-R2.return.json
evidence_ref: process/evidence/ST-AW-003.CP7-R2.index.json
created_at: 2026-07-18T15:09:00Z
context_ref: process/context/stories/STORY-ST-AW-003.CP7-R2.verify-packet.json
story_packet_ref: process/context/stories/STORY-ST-AW-003.CP7-R2.verify-packet.json
dispatch: {required: true, mode: subagent, canonical_role: meta-qa, codex_agent_name: meta-qa-critical, reasoning_profile: critical, dispatch_trigger: CP7-R2-REV-AW003-001-closure, tool_name: followup_task, agent_id: /root/cr051_staw003_qa, thread_id: /root/cr051_staw003_qa, resumed_at: 2026-07-18T15:09:00Z}
question_permission: {can_ask_user: false, mode: relay-via-host-orchestrator, broker_agent: host-orchestrator}
context_policy:
  capsule_first: true
  story_packet_ref: process/context/stories/STORY-ST-AW-003.CP7-R2.verify-packet.json
  read_profile: minimal
  must_read: [delivery/agents/meta-qa.md, process/context/stories/STORY-ST-AW-003.CP7-R2.verify-packet.json, process/docs/quality/CR051-ST-AW-003-REVIEW.md, process/returns/ST-AW-003.CP6-R2.return.json, process/evidence/ST-AW-003.CP6-R2.index.json]
  read_if_needed: [process/stories/STORY-ST-AW-003-heterogeneous-git-legs-IMPLEMENTATION.md（先写 deep_review read-log）]
  do_not_read_by_default: [process/STATE.md, process/DEVELOPMENT-PLAN.yaml, 其他 Story/LLD/IMPLEMENTATION]
---

# ST-AW-003 CP7-R2 独立复验

只复验 REV-AW003-001 的 default-clock freshness 修复及相邻回归；不得修改源码/测试，不得执行真实 remote/worktree/ref/link mutation。输出 R2 四份质量报告与 CP7-R2 return/evidence，不自行写 CP7 result。
