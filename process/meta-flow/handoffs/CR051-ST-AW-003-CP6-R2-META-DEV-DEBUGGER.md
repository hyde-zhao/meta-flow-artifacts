---
handoff_id: H-CR051-ST-AW-003-CP6-R2-META-DEV-DEBUGGER
from_role: host-orchestrator
to_role: meta-dev
canonical_role: meta-dev
workflow_id: meta-flow-self-dev
cr_id: CR-051
story_id: ST-AW-003
stage: story-rework
status: completed
completed_at: 2026-07-18T15:03:00Z
return_ref: process/returns/ST-AW-003.CP6-R2.return.json
evidence_ref: process/evidence/ST-AW-003.CP6-R2.index.json
created_at: 2026-07-18T14:37:00Z
context_ref: process/context/stories/STORY-ST-AW-003.CP6-R2.work-packet.json
story_packet_ref: process/context/stories/STORY-ST-AW-003.CP6-R2.work-packet.json
dispatch: {required: true, mode: subagent, canonical_role: meta-dev, codex_agent_name: meta-dev-debugger, reasoning_profile: debugger, dispatch_trigger: CP7-NEEDS_REWORK-REV-AW003-001-default-clock, tool_name: followup_task, agent_id: /root/cr051_staw003_dev, thread_id: /root/cr051_staw003_dev, resumed_at: 2026-07-18T14:37:00Z}
question_permission: {can_ask_user: false, mode: relay-via-host-orchestrator, broker_agent: host-orchestrator}
context_policy:
  capsule_first: true
  story_packet_ref: process/context/stories/STORY-ST-AW-003.CP6-R2.work-packet.json
  read_profile: minimal
  must_read: [delivery/agents/meta-dev.md, process/context/stories/STORY-ST-AW-003.CP6-R2.work-packet.json, process/docs/quality/CR051-ST-AW-003-FIXES.md, process/docs/quality/CR051-ST-AW-003-REVIEW.md, process/returns/ST-AW-003.CP7.return.json]
  read_if_needed: [process/stories/STORY-ST-AW-003-heterogeneous-git-legs-IMPLEMENTATION.md（先写 deep_review read-log）]
  do_not_read_by_default: [process/STATE.md, process/DEVELOPMENT-PLAN.yaml, 其他 Story/LLD/IMPLEMENTATION]
---

# ST-AW-003 CP6-R2

只关闭 REV-AW003-001：补 now=None 的 source/artifact/ordinary-resume/post-observe 回归，修正 freshness clock-order，保留真正 stale/future skew 的 fail-closed。不得改变 target/authz/publication/recovery 契约，不得改 shared/CLI/aggregate，不得执行真实 publication。更新 IMPLEMENTATION、生成 CP6-R2 return/evidence，不自行批准 CP6。
