---
handoff_id: H-CR051-ST-AW-005-CP6-R2-META-DEV-DEBUGGER
from_role: host-orchestrator
to_role: meta-dev
canonical_role: meta-dev
workflow_id: meta-flow-self-dev
cr_id: CR-051
story_id: ST-AW-005
stage: story-rework
status: completed
completed_at: 2026-07-18T16:27:00Z
return_ref: process/returns/ST-AW-005.CP6-R2.return.json
evidence_ref: process/evidence/ST-AW-005.CP6-R2.index.json
created_at: 2026-07-18T16:11:00Z
context_ref: process/context/stories/STORY-ST-AW-005.CP6-R2.work-packet.json
story_packet_ref: process/context/stories/STORY-ST-AW-005.CP6-R2.work-packet.json
dispatch: {required: true, mode: subagent, canonical_role: meta-dev, codex_agent_name: meta-dev-debugger, reasoning_profile: debugger, dispatch_trigger: CP7-NEEDS_REWORK-REV-AW005-001-denied-descendant, tool_name: followup_task, agent_id: /root/cr051_staw005_dev, thread_id: /root/cr051_staw005_dev, resumed_at: 2026-07-18T16:11:00Z}
question_permission: {can_ask_user: false, mode: relay-via-host-orchestrator, broker_agent: host-orchestrator}
context_policy:
  capsule_first: true
  story_packet_ref: process/context/stories/STORY-ST-AW-005.CP6-R2.work-packet.json
  read_profile: minimal
  must_read: [delivery/agents/meta-dev.md, process/context/stories/STORY-ST-AW-005.CP6-R2.work-packet.json, process/docs/quality/CR051-ST-AW-005-FIXES.md, process/docs/quality/CR051-ST-AW-005-REVIEW.md, process/returns/ST-AW-005.CP7.return.json]
  read_if_needed: [process/stories/STORY-ST-AW-005-read-only-migration-handoff-IMPLEMENTATION.md（先写 deep_review read-log）]
  do_not_read_by_default: [process/STATE.md, process/DEVELOPMENT-PLAN.yaml, process/quant-lab/**, 其他 Story/LLD/IMPLEMENTATION]
---

# ST-AW-005 CP6-R2

只关闭 REV-AW005-001：逐对象 deny guard 必须先于任何 filesystem read/descend/mapping；补宽 root、target deny 和 relative symlink-to-denied 回归。不得改变 schema/CLI/shared/no-follow/O-AW-03，不执行真实 mutation。输出 CP6-R2 return/evidence，不自行写 CP6 result。
