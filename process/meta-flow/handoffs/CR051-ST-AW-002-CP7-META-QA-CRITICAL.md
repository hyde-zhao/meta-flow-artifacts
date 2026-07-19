---
handoff_id: "H-CR051-ST-AW-002-CP7-META-QA-CRITICAL"
from_role: host-orchestrator
to_role: meta-qa
canonical_role: meta-qa
workflow_id: meta-flow-self-dev
cr_id: CR-051
story_id: ST-AW-002
stage: verification-execution
status: completed-via-inline-fallback
completed_at: 2026-07-18T13:05:00Z
created_at: 2026-07-18T12:53:00Z
context_ref: process/context/stories/STORY-ST-AW-002.CP7.verify-packet.json
story_packet_ref: process/context/stories/STORY-ST-AW-002.CP7.verify-packet.json
dispatch:
  required: true
  mode: subagent
  canonical_role: meta-qa
  codex_agent_name: qa-zhang
  reasoning_profile: critical
  dispatch_trigger: CP7-high-risk-CAP-DUR
  tool_name: spawn_agent
  agent_id: /root/cr051_staw002_qa
  thread_id: /root/cr051_staw002_qa
  spawned_at: 2026-07-18T12:54:00Z
  completed_at: 2026-07-18T13:04:00Z
question_permission: {can_ask_user: false, mode: relay-via-host-orchestrator, broker_agent: host-orchestrator}
context_policy:
  capsule_first: true
  story_packet_ref: process/context/stories/STORY-ST-AW-002.CP7.verify-packet.json
  read_profile: minimal
  must_read:
    - delivery/agents/meta-qa.md
    - process/context/stories/STORY-ST-AW-002.CP7.verify-packet.json
    - process/returns/ST-AW-002.CP6.return.json
    - process/evidence/ST-AW-002.CP6.index.json
    - process/docs/features/cr051-worktree/TEST-PLAN.md
  read_if_needed:
    - process/stories/STORY-ST-AW-002-recoverable-project-worktree-IMPLEMENTATION.md（先写 deep_review read-log）
  do_not_read_by_default: [process/STATE.md, process/DEVELOPMENT-PLAN.yaml, 其他 Story/LLD/IMPLEMENTATION]
---

# ST-AW-002 CP7 独立验证

重点独立证明 CAP-01..11、DUR-01..14、WT-01..14、PORT-W-01..08、危险 argv=0 与真实 mutation=0。只验证不改源码/测试。所有 Git/worktree/remote 仅临时 fixture；真实 shared remote/worktree 与 Windows pilot 不得伪报。

必须生成 4 份 `CR051-ST-AW-002-*` 质量报告、CP7 return/evidence 并校验。CAP/DUR 任一不满足直接 `NEEDS_REWORK`，不能风险接受。
