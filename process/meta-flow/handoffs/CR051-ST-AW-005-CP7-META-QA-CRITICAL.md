---
handoff_id: H-CR051-ST-AW-005-CP7-META-QA-CRITICAL
from_role: host-orchestrator
to_role: meta-qa
canonical_role: meta-qa
workflow_id: meta-flow-self-dev
cr_id: CR-051
story_id: ST-AW-005
stage: verification-execution
status: completed-needs-rework-inline-fallback
completed_at: 2026-07-18T16:08:00Z
return_ref: process/returns/ST-AW-005.CP7.return.json
evidence_ref: process/evidence/ST-AW-005.CP7.index.json
created_at: 2026-07-18T15:50:00Z
context_ref: process/context/stories/STORY-ST-AW-005.CP7.verify-packet.json
story_packet_ref: process/context/stories/STORY-ST-AW-005.CP7.verify-packet.json
dispatch: {required: true, mode: subagent, canonical_role: meta-qa, codex_agent_name: meta-qa-critical, reasoning_profile: critical, dispatch_trigger: CP7-read-scope-no-mutation-final-verification, tool_name: spawn_agent, agent_id: /root/cr051_staw005_qa, thread_id: /root/cr051_staw005_qa, spawned_at: 2026-07-18T15:50:00Z}
question_permission: {can_ask_user: false, mode: relay-via-host-orchestrator, broker_agent: host-orchestrator}
context_policy:
  capsule_first: true
  story_packet_ref: process/context/stories/STORY-ST-AW-005.CP7.verify-packet.json
  read_profile: minimal
  must_read: [delivery/agents/meta-qa.md, process/context/stories/STORY-ST-AW-005.CP7.verify-packet.json, process/stories/STORY-ST-AW-005-read-only-migration-handoff.md, process/returns/ST-AW-005.CP6.return.json, process/evidence/ST-AW-005.CP6.index.json, process/docs/features/cr051-migration/TEST-PLAN.md]
  read_if_needed: [process/stories/STORY-ST-AW-005-read-only-migration-handoff-IMPLEMENTATION.md（先写 deep_review read-log）]
  do_not_read_by_default: [process/STATE.md, process/DEVELOPMENT-PLAN.yaml, process/quant-lab/**, 其他 Story/LLD/IMPLEMENTATION]
---

# ST-AW-005 CP7 独立验证

独立证明只读 manifest/handoff 的 portable、fail-closed、read-scope 与零 mutation 契约。只验证，不改源码/测试，不执行真实迁移/remote/worktree/ref/link。生成四份质量报告与 CP7 return/evidence，不自行写 CP7 result。
