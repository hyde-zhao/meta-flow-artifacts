---
handoff_id: "HO-CR051-CP4-META-SE-FEAT345"
from_agent: "host-orchestrator"
to_agent: "meta-se"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "story-planning"
status: "completed"
created_at: "2026-07-18T06:12:00Z"
context_ref: "process/context/CP4-CR051-STORY-PLANNING-CONTEXT.yaml"
write_lane: "feature-packs-03-05"
dispatch:
  mode: "subagent"
  canonical_role: "meta-se"
  codex_agent_name: "meta-se-critical"
  reasoning_profile: "critical"
  dispatch_trigger: "CP4 split lane for FEAT-AW-03/04/05 after bounded-window stall"
  tool_name: "followup_task"
  agent_id: "/root/cr051_meta_se_cp3_finish"
  thread_id: "/root/cr051_meta_se_cp3_finish"
  resumed_at: "2026-07-18T06:14:16Z"
  completed_at: "2026-07-18T06:39:13Z"
---

# CR-051 CP4 Meta-SE FEAT-AW-03/04/05 Lane

完整遵循 `implementation-design` 和模板；读取已批准 Blueprint/HLD/ADR、Domain/Dependency Map 与 CP4 capsule。

仅写：

- `process/docs/features/cr051-legs/{DESIGN,TEST-PLAN,TASKS}.md`
- `process/docs/features/cr051-aggregate/{DESIGN,TEST-PLAN,TASKS}.md`
- `process/docs/features/cr051-migration/{DESIGN,TEST-PLAN,TASKS}.md`
- `process/handoffs/CR051-CP4-META-SE-FEAT345-RETURN.md`

FEAT-AW-03 固定异构 source-default/artifact-integration，禁止 artifact main refresh；FEAT-AW-04 固定单写聚合与 `BLOCKED > FAIL > IN_PROGRESS > PASS`，PARTIAL 非成功；FEAT-AW-05 只读 migration manifest/handoff，真实 mutation=0，并消费 O-AW-03 指标与三个阈值，阈值只建 follow-up candidate。不得写 core planning、其他 Feature、CP4 result、源码/测试/STATE/ledger。
