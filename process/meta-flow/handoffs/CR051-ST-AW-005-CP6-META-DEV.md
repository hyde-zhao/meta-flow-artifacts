---
handoff_id: H-CR051-ST-AW-005-CP6-META-DEV
from_role: host-orchestrator
to_role: meta-dev
canonical_role: meta-dev
workflow_id: meta-flow-self-dev
cr_id: CR-051
story_id: ST-AW-005
stage: story-execution
status: completed
completed_at: 2026-07-18T15:47:00Z
return_ref: process/returns/ST-AW-005.CP6.return.json
evidence_ref: process/evidence/ST-AW-005.CP6.index.json
created_at: 2026-07-18T15:22:00Z
context_ref: process/context/stories/STORY-ST-AW-005.CP6.work-packet.json
story_packet_ref: process/context/stories/STORY-ST-AW-005.CP6.work-packet.json
dispatch: {required: true, mode: subagent, canonical_role: meta-dev, codex_agent_name: dev-yang, reasoning_profile: default, dispatch_trigger: W4-read-only-migration-handoff, tool_name: spawn_agent, agent_id: /root/cr051_staw005_dev, thread_id: /root/cr051_staw005_dev, spawned_at: 2026-07-18T15:22:00Z}
question_permission: {can_ask_user: false, mode: relay-via-host-orchestrator, broker_agent: host-orchestrator}
context_policy:
  capsule_first: true
  story_packet_ref: process/context/stories/STORY-ST-AW-005.CP6.work-packet.json
  read_profile: minimal
  must_read: [delivery/agents/meta-dev.md, process/context/stories/STORY-ST-AW-005.CP6.work-packet.json, process/stories/STORY-ST-AW-005-read-only-migration-handoff.md, process/docs/features/cr051-migration/TEST-PLAN.md, process/returns/ST-AW-003.CP7-R2.return.json, process/returns/ST-AW-004.CP7.return.json]
  read_if_needed: [process/docs/features/cr051-migration/DESIGN.md, process/docs/features/cr051-migration/TASKS.md]
  do_not_read_by_default: [process/STATE.md, process/DEVELOPMENT-PLAN.yaml, process/quant-lab/**, 其他 Story/LLD/IMPLEMENTATION]
---

# ST-AW-005 CP6

实现只读 migration preflight、11 分区 portable manifest 与 O-AW-03 candidate-only evaluator。目标项目、sibling、Git/worktree/ref/remote/link 的真实 mutation 必须为 0；不得接线 CLI，不得生成可执行迁移脚本。输出 IMPLEMENTATION、CP6 return/evidence，不自行写 CP6 result。
