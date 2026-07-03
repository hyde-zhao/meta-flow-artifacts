---
handoff_id: "H-CR037-CP5-LLD-BATCH-B"
created_at: "2026-07-03T00:00:00+08:00"
from_agent: "host-orchestrator"
to_agent: "meta-dev"
workflow_id: "meta-flow"
change_id: "CR-037"
batch_id: "CR037-CP5-LLD-B"
status: "completed"
dispatch:
  required: true
  semantic: "stage-dispatch"
  mode: "subagent"
  platform: "codex"
  agent_role: "meta-dev"
  canonical_role: "meta-dev"
  codex_agent_name: "meta-dev"
  reasoning_profile: "default"
  dispatch_trigger: "cp5-lld-batch"
  agent_path: ""
  tool_name: "spawn_agent"
  agent_id: "019f2588-e0c2-7dc1-a8a4-8c14b58bfecc"
  agent_name: "dev-shi"
  thread_id: "019f2588-e0c2-7dc1-a8a4-8c14b58bfecc"
  spawned_at: "2026-07-03T00:00:00+08:00"
  resumed_at: ""
  completed_at: "2026-07-03T00:00:00+08:00"
  evidence: "spawn_agent"
  fallback_reason: ""
  approved_by: "user"
  approved_at: "2026-07-03T00:00:00+08:00"
question_permission:
  can_ask_user: false
  mode: "queue-only"
  structured_choice_allowed: false
  allowed_question_scope: "Only write clarification candidates into assigned design evidence; host-orchestrator brokers user questions."
  forbidden_question_scope: "CP2/CP3/CP5/CP8 formal gates, runtime authorization, credentials, security boundary changes, publish, live/trading authorization"
  broker_agent: "host-orchestrator"
context_policy:
  capsule_first: true
  capsule_path: "process/context/CP5-CR-037-LLD-CONTEXT.yaml"
  context_ref: "process/context/CP5-CR-037-LLD-CONTEXT.yaml"
  story_packet_ref: ""
  read_profile: "compact"
  max_source_files: 16
  full_doc_read_policy: "only-on-missing-conflict-audit-or-deep-review"
  full_doc_read_reason: ""
  allowed_reads:
    - "process/context/CP5-CR-037-LLD-CONTEXT.yaml"
    - "process/stories/STORY-CR037-S04-ledger-compaction-policy-and-cli.md"
    - "process/stories/STORY-CR037-S05-project-scaffold-and-project-current.md"
    - "process/stories/STORY-CR037-S06-project-scale-and-roadmap-objects.md"
    - "process/docs/features/ledger-compaction/DESIGN.md"
    - "process/docs/features/ledger-compaction/TEST-PLAN.md"
    - "process/docs/features/ledger-compaction/TASKS.md"
    - "process/docs/features/project-state-governance/DESIGN.md"
    - "process/docs/features/project-state-governance/TEST-PLAN.md"
    - "process/docs/features/project-state-governance/TASKS.md"
  must_read:
    - "process/context/CP5-CR-037-LLD-CONTEXT.yaml"
    - "process/stories/STORY-CR037-S04-ledger-compaction-policy-and-cli.md"
    - "process/stories/STORY-CR037-S05-project-scaffold-and-project-current.md"
    - "process/stories/STORY-CR037-S06-project-scale-and-roadmap-objects.md"
  read_if_needed:
    - "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
    - "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
    - "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
    - "meta_flow/cli.py"
    - "meta_flow/state/current.py"
    - "meta_flow/workspace/routing.py"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "process/quant-lab/**"
    - "process/stories/*-LLD.md"
---

# CR037 CP5 LLD Batch B Handoff

## Assigned Stories

- `CR037-S04` ledger compaction policy and CLI：full-lld，输出 `process/stories/STORY-CR037-S04-ledger-compaction-policy-and-cli-LLD.md`。
- `CR037-S05` project scaffold and PROJECT.current：full-lld，输出 `process/stories/STORY-CR037-S05-project-scaffold-and-project-current-LLD.md`。
- `CR037-S06` PROJECT-SCALE and roadmap objects：full-lld，输出 `process/stories/STORY-CR037-S06-project-scale-and-roadmap-objects-LLD.md`。

## Write Scope

只允许写上述三个 LLD 文件。不得修改代码、测试、STATE、ledger、CR index、其他 Story 或 Feature 设计。

## Exit Criteria

- full-lld 覆盖 lld-designer 要求的 14 章节。
- 明确 `state compact` 与 `ledger compact` 职责边界、PROJECT.current refs-only 约束、project-scale 与 gate_profile bias 的关系。
- 若存在 blocks_lld 问题，只写 clarification candidate，不直接问用户。
