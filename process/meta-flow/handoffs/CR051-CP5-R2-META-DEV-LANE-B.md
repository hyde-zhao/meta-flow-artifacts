---
handoff_id: "HO-CR051-CP5-R2-META-DEV-LANE-B"
from_agent: "host-orchestrator"
to_agent: "meta-dev"
workflow_id: "meta-flow-self-dev"
change_id: "CR-051"
stage: "story-design-revision"
status: "returned"
created_at: "2026-07-18T07:28:00Z"
context_ref: "process/context/CP5-CR051-LLD-CONTEXT.yaml"
story_ids: ["ST-AW-002"]
finding_refs: ["CP5-QA-R1-F02"]
dispatch:
  mode: "subagent"
  canonical_role: "meta-dev"
  codex_agent_name: "meta-dev-debugger"
  reasoning_profile: "debugger"
  dispatch_trigger: "CP5 R1 independent review F02: WorktreeHealth/Observation cross-Story port mismatch"
  tool_name: "followup_task"
  agent_id: "/root/cr051_meta_se_cp3"
  thread_id: "/root/cr051_meta_se_cp3"
  resumed_at: "2026-07-18T07:29:00Z"
  completed_at: "2026-07-18T07:41:05Z"
---

# CR-051 CP5 R2 Lane B — Worktree Health Port

只处理 ST-AW-002 一侧的 `CP5-QA-R1-F02`。公共契约固定为：

- `observe_worktree(...) -> WorktreeObservation`，rich immutable snapshot 至少含 project/repo/worktree identity、common-dir、HEAD/ref/OID、dirty/staged/untracked、Git-op、registry/role、observed_at、digest；unknown 是显式值并 fail closed。
- `evaluate_worktree_health(observation, journal_state, active_operation_id, ...) -> WorktreeHealth`；`WorktreeHealth` 必含 `project_id`、decision、`observation: WorktreeObservation | None`、`observation_digest`、worktree/journal state、active operation、reason codes。
- `HEALTHY` 必须有非空 observation 且 digest 精确一致；非 HEALTHY 可保留 observation，但不得授权 mutation。
- ST-AW-003 只能读取 `health.observation`，不得把 rich snapshot 字段平铺为另一套 Health schema。
- 不弱化 O-AW-01/02、CAP/DUR/WT fixture 或 Git-before-durable=0。

仅允许修改/新建：

- `process/docs/features/cr051-worktree/DESIGN.md`
- `process/docs/features/cr051-worktree/TEST-PLAN.md`
- `process/stories/STORY-ST-AW-002-recoverable-project-worktree-LLD.md`
- `process/stories/STORY-ST-AW-002-recoverable-project-worktree.md`
- `process/checks/CP5-CR051-ST-AW-002-LLD-IMPLEMENTABILITY-R2.result.json`
- `process/checks/CP5-CR051-ST-AW-002-LLD-IMPLEMENTABILITY-R2.result.summary.md`
- `process/handoffs/CR051-CP5-R2-META-DEV-LANE-B-RETURN.md`

R2 result/检查要求同 Lane A。不得改 ST-AW-003/其他共享对象/源码/测试/plan/status/state/ledger/checkpoint；return 中完整复述 frozen public port，供 Lane C 对齐。
