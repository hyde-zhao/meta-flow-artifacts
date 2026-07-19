---
handoff_id: "HO-CR051-CP5-R2-META-DEV-LANE-A"
from_agent: "host-orchestrator"
to_agent: "meta-dev"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "story-design-revision"
status: "returned"
created_at: "2026-07-18T07:28:00Z"
context_ref: "process/context/CP5-CR051-LLD-CONTEXT.yaml"
story_ids: ["ST-AW-001"]
finding_refs: ["CP5-QA-R1-F01"]
dispatch:
  mode: "subagent"
  canonical_role: "meta-dev"
  codex_agent_name: "meta-dev"
  reasoning_profile: "default"
  dispatch_trigger: "CP5 R1 independent review F01: PathRef anchor schema contradiction"
  tool_name: "followup_task"
  agent_id: "/root/cr051_cp4_core"
  thread_id: "/root/cr051_cp4_core"
  resumed_at: "2026-07-18T07:29:00Z"
  completed_at: "2026-07-18T07:44:45Z"
---

# CR-051 CP5 R2 Lane A — Routing Anchor Schema

只处理 `CP5-QA-R1-F01`。冻结以下修复：

- `PathRef.anchor` 枚举必须显式包含 `project_worktree`；
- anchor DAG 固定为 `project_root → {artifact_control_root, sibling_root} → project_worktree → {docs_relative, process_relative}`，legacy docs/process 仍从 `artifact_control_root`；
- schema/loader 必须拒绝未知 anchor、错误父 anchor、环、绝对路径、`.`/`..`、越界解析；
- portable/canonical digest 只包含 anchor+relative path，不包含设备绝对路径；
- 不改变 project-first/legacy 路由边界、owned target 或 Git mutation=0。

仅允许修改/新建：

- `process/docs/features/cr051-routing/DESIGN.md`
- `process/docs/features/cr051-routing/TEST-PLAN.md`
- `process/stories/STORY-ST-AW-001-project-first-routing-LLD.md`（版本+修订记录）
- `process/stories/STORY-ST-AW-001-project-first-routing.md`（R2 result ref/ready-for-review）
- `process/checks/CP5-CR051-ST-AW-001-LLD-IMPLEMENTABILITY-R2.result.json`
- `process/checks/CP5-CR051-ST-AW-001-LLD-IMPLEMENTABILITY-R2.result.summary.md`
- `process/handoffs/CR051-CP5-R2-META-DEV-LANE-A-RETURN.md`

R2 result 必须 `supersedes_result_ref` 指向 R1，记录真实 read-expansion ref，`lld-check` 和 `cp result-check --check-consistency --correlation-profile audit` 均 OK。不得改共享 plan/status/state/ledger/checkpoint/其他 Story/源码/测试；clarification 只写 return。
