---
handoff_id: H-CR051-ST-AW-004-CP6-META-DEV
from_role: host-orchestrator
to_role: meta-dev
canonical_role: meta-dev
workflow_id: meta-flow-self-dev
cr_id: CR-051
story_id: ST-AW-004
stage: story-execution
status: completed
created_at: 2026-07-18T13:08:00Z
context_ref: process/context/stories/STORY-ST-AW-004.CP6.work-packet.json
story_packet_ref: process/context/stories/STORY-ST-AW-004.CP6.work-packet.json
dispatch: {required: true, mode: subagent, canonical_role: meta-dev, codex_agent_name: dev-qin, reasoning_profile: default, dispatch_trigger: W3-aggregate-projection, tool_name: spawn_agent, agent_id: /root/cr051_staw004_dev, thread_id: /root/cr051_staw004_dev, spawned_at: 2026-07-18T13:09:00Z}
completed_at: 2026-07-18T14:11:00Z
return_ref: process/returns/ST-AW-004.CP6.return.json
evidence_ref: process/evidence/ST-AW-004.CP6.index.json
question_permission: {can_ask_user: false, mode: relay-via-host-orchestrator, broker_agent: host-orchestrator}
context_policy:
  capsule_first: true
  story_packet_ref: process/context/stories/STORY-ST-AW-004.CP6.work-packet.json
  read_profile: minimal
  must_read: [delivery/agents/meta-dev.md, process/context/stories/STORY-ST-AW-004.CP6.work-packet.json, process/docs/features/cr051-aggregate/DESIGN.md, process/docs/features/cr051-aggregate/TEST-PLAN.md]
  read_if_needed: [process/stories/STORY-ST-AW-004-aggregate-evidence-gate-LLD.md（实现前 deep_review read-log）, meta_flow/cli.py, meta_flow/workflow/cr_lifecycle.py, meta_flow/state/current.py]
---

# ST-AW-004 CP6

实现 validated published handles 的 16 组合纯聚合、single-writer persist/readback、conflict/idempotency 和仅 2/2 PASS controlled projection；本 Story 是 CLI/shared projection 单写 owner。不得读取或修改 ST-AW-003 primary，不得 import/call Git/worktree，不得为 non-PASS 自动 close/rollback。生成 IMPLEMENTATION/return/evidence，不批准 CP6。
