---
handoff_id: "H-CR037-CP5-LLD-BATCH-D"
created_at: "2026-07-03T00:00:00+08:00"
from_agent: "host-orchestrator"
to_agent: "meta-dev"
workflow_id: "meta-flow"
change_id: "CR-037"
batch_id: "CR037-CP5-LLD-D"
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
  agent_id: "019f2589-4960-7700-9bc2-be0d06e6a07a"
  agent_name: "dev-lv"
  thread_id: "019f2589-4960-7700-9bc2-be0d06e6a07a"
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
    - "process/stories/STORY-CR037-S11-fu-rf-tracking-support.md"
    - "process/stories/STORY-CR037-S12-project-stale-check.md"
    - "process/stories/STORY-CR037-S13-quant-lab-migration-dry-run-and-reports.md"
    - "process/docs/features/roadmap-follow-up-tracking/DESIGN.md"
    - "process/docs/features/project-stale-check/DESIGN.md"
    - "process/docs/features/quant-lab-migration-readiness/DESIGN.md"
  must_read:
    - "process/context/CP5-CR-037-LLD-CONTEXT.yaml"
    - "process/stories/STORY-CR037-S11-fu-rf-tracking-support.md"
    - "process/stories/STORY-CR037-S12-project-stale-check.md"
    - "process/stories/STORY-CR037-S13-quant-lab-migration-dry-run-and-reports.md"
  read_if_needed:
    - "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-HLD.md"
    - "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-ARCHITECTURE-DECISION.md"
    - "process/docs/design/META-FLOW-PROJECT-GOVERNANCE-FEATURE-DESIGN-MATRIX.md"
    - "process/docs/features/roadmap-follow-up-tracking/TEST-PLAN.md"
    - "process/docs/features/project-stale-check/TEST-PLAN.md"
    - "process/docs/features/quant-lab-migration-readiness/TEST-PLAN.md"
    - "process/quant-lab/**"
  do_not_read_by_default:
    - "process/STATE.md"
    - "process/DEVELOPMENT-PLAN.yaml"
    - "quant-lab release repo writes"
    - "credentials"
    - "runtime"
    - "publish"
    - "live"
    - "production write"
---

# CR037 CP5 LLD Batch D Handoff

## Assigned Stories

- `CR037-S11` FU-RF tracking support：technical-note，更新 Story 卡片 `## 技术说明`。
- `CR037-S12` project stale-check：full-lld，输出 `process/stories/STORY-CR037-S12-project-stale-check-LLD.md`。
- `CR037-S13` quant-lab migration dry-run and reports：full-lld，输出 `process/stories/STORY-CR037-S13-quant-lab-migration-dry-run-and-reports-LLD.md`。

## Write Scope

只允许写上述 Story 卡片和上述两个 LLD 文件。不得修改代码、测试、STATE、ledger、CR index、其他 Story 或 Feature 设计。`process/quant-lab/**` 仅允许按需只读；不得写 quant-lab 发布库或执行 runtime / publish / live。

## Exit Criteria

- full-lld 覆盖 lld-designer 要求的 14 章节。
- S13 必须处理 `R-CR037-S13-LONG-CHAIN`，写明上游延期时 blocked、拆分或缩窄策略。
- 若存在 blocks_lld 问题，只写 clarification candidate，不直接问用户。
