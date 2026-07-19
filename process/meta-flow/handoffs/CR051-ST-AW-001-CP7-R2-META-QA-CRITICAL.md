---
handoff_id: "H-CR051-ST-AW-001-CP7-R2-META-QA-CRITICAL"
from_role: "host-orchestrator"
to_role: "meta-qa"
canonical_role: "meta-qa"
workflow_id: "meta-flow-self-dev"
cr_id: "CR-051"
story_id: "ST-AW-001"
wave_id: "QW1-R2"
stage: "verification-reexecution"
status: "completed-via-inline-fallback"
completed_at: "2026-07-18T12:35:00Z"
created_at: "2026-07-18T12:23:00Z"
context_ref: "process/context/stories/STORY-ST-AW-001.CP7-R2.verify-packet.json"
story_packet_ref: "process/context/stories/STORY-ST-AW-001.CP7-R2.verify-packet.json"
dispatch:
  required: true
  mode: "subagent"
  platform: "codex"
  canonical_role: "meta-qa"
  codex_agent_name: "qa-shi"
  reasoning_profile: "critical"
  dispatch_trigger: "CP7-R2-finding-closure"
  tool_name: "spawn_agent"
  agent_id: "/root/cr051_staw001_qa_r2"
  thread_id: "/root/cr051_staw001_qa_r2"
  spawned_at: "2026-07-18T12:24:00Z"
  completed_at: "2026-07-18T12:34:00Z"
question_permission:
  can_ask_user: false
  mode: "relay-via-host-orchestrator"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  story_packet_ref: "process/context/stories/STORY-ST-AW-001.CP7-R2.verify-packet.json"
  read_profile: "minimal"
  must_read:
    - "delivery/agents/meta-qa.md"
    - "process/context/stories/STORY-ST-AW-001.CP7-R2.verify-packet.json"
    - "process/returns/ST-AW-001.CP6-R2.return.json"
    - "process/evidence/ST-AW-001.CP6-R2.index.json"
    - "process/docs/quality/CR051-ST-AW-001-FIXES.md"
  read_if_needed:
    - "process/stories/STORY-ST-AW-001-project-first-routing-IMPLEMENTATION.md（先写 deep_review read-log）"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "其他 Story / LLD / IMPLEMENTATION"
---

# ST-AW-001 CP7-R2 独立重验

只重验 `CP7-AW-001-F01/F02` closure，并回归完整 38 项。QA 不得修改源码/测试，不得执行真实 Git/worktree/ref/remote/link/migration。

必须交付：

- 四份 `CR051-ST-AW-001-*-R2.md` 质量报告；无新修复时 FIXES-R2 明确 N/A。
- `process/returns/ST-AW-001.CP7-R2.return.json` 与 `process/evidence/ST-AW-001.CP7-R2.index.json`。
- 结论 `PASS` 或 `PASS_WITH_RISK` 仅在 F01/F02 均独立关闭、38 tests 与静态检查通过时允许；否则返回 `NEEDS_REWORK`。
- module manifest 与真实跨设备 pilot 继续按实际证据记录，禁止伪报 PASS。
